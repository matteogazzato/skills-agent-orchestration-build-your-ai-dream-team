# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent custom team defined under `.github/agents/`, orchestrated with GitHub Copilot CLI in a Codespace.

| Agent | Model | Responsibility | Definition |
|---|---|---|---|
| Orchestrator | Claude Opus 4.7 (copilot) | Coordinates the Planner, Coder, and Designer agents. Breaks requests into phases, assigns explicit file scopes, runs non-overlapping work in parallel, and reports the final outcome. Does not implement work itself. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the repository and relevant docs/dependencies, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable vs. sequential work, edge cases, and open questions. Does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code, fixes bugs, and builds runnable-app support (e.g., `.vscode/launch.json` configured for the `app` folder) within the file scope assigned by the Orchestrator. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Owns UI/UX, accessibility, and visual design for the dashboard—project cards, status badges, priority treatment, and responsive layout with deterministic CSS hooks like `.dashboard` and `.project-card`. | `.github/agents/designer.agent.md` |

I'm using GitHub Copilot CLI running in a Codespace to orchestrate this team: the Orchestrator delegates to Planner, Coder, and Designer, and I control all git operations manually through Copilot CLI prompts.
