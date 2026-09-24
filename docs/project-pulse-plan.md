# Mona's Project Pulse Dashboard — Implementation Plan

## 1. Summary

Build a small, static, self-contained Project Pulse dashboard under `app/` that renders project cards from a local JSON data file, plus a VS Code launch configuration that previews it. Work is split across two agents: the **Designer** owns visual structure/markup and CSS (`app/index.html`, `app/styles.css`), and the **Coder** owns data, JS wiring, and tooling (`app/project-data.json`, `app/app.js`, `.vscode/launch.json`). The dashboard must open directly to `index.html` (not a directory listing) with `cwd` set to `${workspaceFolder}/app`, per `coder.agent.md`. Visual output must include `.dashboard` and `.project-card` CSS hooks, status badges, priority treatment, and a responsive layout, per `designer.agent.md`.

The brief requires a top-level `projects` array with `name`, `owner`, `status`, `recentActivity`, and `priority`. The task also mentions `due date`, so `dueDate` is treated as an optional additional field so both requirements are satisfied without breaking the core schema.

Since the existing `.vscode/tasks.json` only defines an unrelated exercise-terminal task, the launch config is greenfield and does not need to integrate with any preLaunchTask unless a static server is required (see Open Questions).

---

## 2. Ordered implementation steps

1. **Contract fixed** (Planner, this document): lock the JSON schema, DOM contract, CSS hook names, and launch config shape so Designer + Coder can proceed in parallel without collisions.
2. **Author sample data** — `app/project-data.json` with a top-level `projects` array of 5–7 realistic entries covering every status and priority value at least once.
3. **Design markup skeleton** — `app/index.html` shell containing header, `<main class="dashboard">`, an empty `<section id="projects" class="dashboard__grid">` mount point, an empty-state element, and a `<template id="project-card-template">` matching the agreed card DOM contract. Link `styles.css` and `app.js` (deferred).
4. **Design visual styling** — `app/styles.css` implementing `.dashboard`, `.project-card`, `.project-card__header`, status badge classes (`.badge`, `.badge--active`, `.badge--blocked`, `.badge--on-hold`, `.badge--done`), priority classes (`.priority--high|medium|low`), typography, spacing, responsive grid (CSS Grid with `auto-fill, minmax(280px, 1fr)`), rounded corners, shadows, focus states, and reduced-motion respect.
5. **Implement data wiring** — `app/app.js` that `fetch`es `./project-data.json`, clones the `<template>` per project, populates fields, applies badge/priority modifier classes derived from data, formats `dueDate` if present, and renders an empty/error state on failure.
6. **Create launch config** — `.vscode/launch.json` with a configuration named **Run Project Pulse Dashboard**, opening `index.html` with `cwd: ${workspaceFolder}/app`.
7. **Integration validation** — open via the launch config, confirm cards render, badges/priorities style correctly, responsive behavior works, and error/empty states behave.

---

## 3. File assignments

| # | File | Owner | Purpose / expected contents |
|---|---|---|---|
| 1 | `app/index.html` | **Designer** | Semantic HTML5 doc: `<header>` with dashboard title + subtitle, `<main class="dashboard">`, `<section class="dashboard__grid" id="projects">` mount point, `<p class="dashboard__empty" hidden>` empty state, `<template id="project-card-template">` with `.project-card`, `.project-card__title`, `.project-card__owner`, `.badge` (status), `.priority`, `.project-card__activity`, `.project-card__due` slots. Loads `styles.css` in `<head>` and `app.js` with `defer`. |
| 2 | `app/styles.css` | **Designer** | All visual styling. Must define `.dashboard`, `.dashboard__grid` (responsive CSS Grid), `.project-card` (padding, radius, shadow, border), status badge modifiers, priority treatment (color/left border/icon), typography scale, color tokens via `:root` custom properties, focus-visible outlines, small-viewport breakpoint. No JS, no data. |
| 3 | `app/project-data.json` | **Coder** | `{ "projects": [ { "name", "owner", "status", "recentActivity", "priority", "dueDate" }, ... ] }`. 5–7 entries. `status` ∈ {`active`, `blocked`, `on-hold`, `done`}; `priority` ∈ {`high`, `medium`, `low`}; `dueDate` ISO `YYYY-MM-DD`. Strict JSON, no comments/trailing commas. |
| 4 | `app/app.js` *(added by Planner — required to wire data → DOM without inline scripts)* | **Coder** | Fetches `./project-data.json`, validates `projects` is an array, iterates and clones template, sets textContent for each slot, adds `badge--${status}` and `priority--${priority}` classes, formats `dueDate` with `Intl.DateTimeFormat`, shows empty state when array is empty, shows error state on fetch/parse failure. No frameworks. |
| 5 | `.vscode/launch.json` | **Coder** | Strict JSON. One configuration named `Run Project Pulse Dashboard`. Uses built-in `type: "chrome"` or `"msedge"` with `request: "launch"`, `url` pointing at `index.html` served from the app folder, and `cwd: "${workspaceFolder}/app"` per `coder.agent.md`. Alternative (see Open Questions): a `node` type running a static server task with the same `cwd`. |

Ownership rule enforcement: Designer must not touch `project-data.json`, `app.js`, or `.vscode/launch.json`. Coder must not restyle `styles.css` or restructure the visual markup in `index.html` (Coder only reads it to align DOM contract).

---

## 4. Dependencies between steps

- Step 1 (contract) blocks Steps 2–6.
- Step 5 (`app.js`) depends on Step 3 (`index.html` template DOM) **and** Step 2 (`project-data.json` schema).
- Step 4 (`styles.css`) depends on Step 3 (class names in markup).
- Step 6 (`launch.json`) depends only on Step 3 existing at the expected path (`app/index.html`).
- Step 7 (validation) depends on all prior steps.

---

## 5. Work that can run in parallel

After Step 1 (contract locked):

- **Designer track**: Step 3 (`index.html`) → Step 4 (`styles.css`).
- **Coder track**: Step 2 (`project-data.json`) and Step 6 (`.vscode/launch.json`) can start immediately in parallel with the Designer track. Step 5 (`app.js`) can be drafted in parallel using the agreed template DOM contract and finalized once Step 3 lands.

---

## 6. Work that must run sequentially

- Step 1 before everything else.
- Step 3 before Step 4 (CSS needs final class names) and before final commit of Step 5 (JS needs final template IDs/class hooks).
- Step 2 before final commit of Step 5 (JS depends on JSON keys).
- Step 7 last.

---

## 7. Edge cases to handle

- **Empty `projects` array** → show `.dashboard__empty` message ("No active projects yet"), do not render an empty grid.
- **Fetch failure / malformed JSON** → visible error state with actionable text; log to console; never leave a blank page.
- **Missing optional fields** (`dueDate`, `recentActivity`) → hide the corresponding element rather than rendering "undefined".
- **Unknown `status` or `priority` values** → fall back to a neutral badge/priority class; do not throw.
- **Long project names / owner strings** → CSS must handle overflow (ellipsis or wrap) without breaking card layout.
- **Date formatting** → use `Intl.DateTimeFormat` with a fixed locale (e.g., `en-US`) for deterministic output; guard against invalid dates.
- **File:// vs http:// serving** → `fetch('./project-data.json')` will fail under `file://` in some browsers. Launch config MUST serve over HTTP (see Open Questions).
- **Cwd correctness** — Coder must verify `cwd` is `${workspaceFolder}/app` (per `coder.agent.md`) and the opened URL resolves to `index.html`, not a directory listing.
- **Accessibility** — badges must not rely on color alone (include text label); card must be reachable/readable by screen readers; sufficient contrast on badge colors.
- **Responsive layout** — grid collapses to single column below ~480px; header remains readable.
- **Reduced motion** — any hover/transition respects `prefers-reduced-motion`.

---

## 8. Validation expectations

- `.vscode/launch.json` parses as strict JSON; the **Run Project Pulse Dashboard** configuration appears in VS Code Run panel.
- Launching opens `index.html` (not a directory index), with `cwd` = `${workspaceFolder}/app`.
- `app/project-data.json` parses (e.g., `python -m json.tool` or `jq .`) and matches the agreed schema.
- Dashboard renders one `.project-card` per entry in `projects`.
- Every card shows name, owner, a status `.badge`, a priority indicator, recent activity, and (if present) formatted due date.
- Removing all entries from `projects` shows the empty state; corrupting the JSON shows the error state.
- Layout reflows at narrow widths without horizontal scroll.
- No console errors on load.
- Designer confirms `.dashboard` and `.project-card` hooks are present and visually distinct, per `designer.agent.md`.
- Coder confirms no design-only files were modified, per `coder.agent.md` rules.

---

## 9. Open questions

1. **Launch type**: Should `.vscode/launch.json` use `chrome`/`msedge` debug (requires the extension and a running static server) or a Node-based static server (e.g., `npx serve app`) invoked via `preLaunchTask`? Serving the app implies HTTP. Recommendation: `chrome` launch with `url: http://localhost:8080/index.html` plus a `preLaunchTask` added to `.vscode/tasks.json` that runs `npx --yes http-server -p 8080 .` with `cwd: ${workspaceFolder}/app`. Confirm before Coder finalizes Step 6 — this may require appending a task to the existing `tasks.json`.
2. **`dueDate` inclusion**: The brief schema lists five fields; the task adds "due date." Plan includes `dueDate` as optional. Confirm this is desired.
3. **Status vocabulary**: Confirm the allowed values (`active`, `blocked`, `on-hold`, `done`) — these drive Designer's badge palette.
4. **Priority vocabulary**: Confirm `high` / `medium` / `low`.
5. **Branding**: Any color palette, logo, or product name styling Mona wants beyond "Project Pulse"?
6. **Sample data authenticity**: Should sample projects reference real team/repo context, or generic placeholders?
7. **Framework stance**: Confirm plain HTML/CSS/vanilla JS (no build step, no npm dependency for the app itself) — assumed here.
8. **Adding `app/app.js`**: The brief lists three `app/` files; this plan adds a fourth (`app.js`) because inline scripts would violate the Designer/Coder file-ownership split. Confirm acceptable, or alternatively allow Coder to own a `<script>` block inside `index.html` (which would require joint ownership of that file — not recommended).
