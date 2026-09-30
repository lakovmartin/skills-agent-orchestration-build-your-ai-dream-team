# Agent team

To build Mona's Project Pulse dashboard, I am using a four-agent custom team defined under `.github/agents/`:

- **Orchestrator** (model: Claude Opus 4.7 (copilot)) — Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases, assigns explicit file scopes to avoid overlap, decides what can run in parallel vs. sequentially, and reports the integrated result. Defined in `.github/agents/orchestrator.agent.md`.
- **Planner** (model: Claude Opus 4.7 (copilot)) — Researches the repository and relevant docs/dependencies, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, edge cases, and validation expectations. Does not write code. Defined in `.github/agents/planner.agent.md`.
- **Coder** (model: GPT-5.5 (copilot)) — Implements the dashboard logic and any supporting configuration (e.g., `.vscode/launch.json`) within the file scope assigned by the Orchestrator, following repository patterns and validating changes before reporting completion. Defined in `.github/agents/coder.agent.md`.
- **Designer** (model: Gemini 3.1 Pro (copilot)) — Owns UI/UX for the dashboard: project cards, status badges, priority treatment, responsive layout, and deterministic CSS hooks (`.dashboard`, `.project-card`), ensuring a polished, accessible interface. Defined in `.github/agents/designer.agent.md`.

All four agents are orchestrated using GitHub Copilot CLI running in a Codespace; the learner (Mona) retains control of all git operations, as none of the agents stage, commit, or push changes themselves.
