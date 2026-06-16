# Agent team

This project uses four custom agents defined under `.github/agents/` to build Mona's Project Pulse dashboard. The work is orchestrated interactively through **GitHub Copilot CLI running inside a Codespace**.

| Agent | Model | Definition | Responsibility |
|---|---|---|---|
| **Orchestrator** | Claude Opus 4.7 | `.github/agents/orchestrator.agent.md` | Coordinates the other agents. Breaks the request into phases, assigns file scopes, runs tasks in parallel where safe, and reports the final outcome. Does not write code itself. |
| **Planner** | Claude Opus 4.7 | `.github/agents/planner.agent.md` | Researches the codebase and produces a detailed implementation plan — ordered steps, file assignments, dependencies, parallelism opportunities, edge cases, and validation expectations. Does not write code. |
| **Coder** | GPT-5.5 | `.github/agents/coder.agent.md` | Implements code, fixes bugs, and produces testable logic within the file scope assigned by the Orchestrator. Also creates support files such as `.vscode/launch.json` for the runnable dashboard app. |
| **Designer** | Gemini 3.1 Pro | `.github/agents/designer.agent.md` | Owns UI/UX, accessibility, visual design, and responsive layout. Produces polished dashboard styling — project cards, status badges, priority treatment, and CSS hooks like `.dashboard` and `.project-card`. |

> **Orchestration environment:** All agent coordination is performed via **GitHub Copilot CLI** in a **GitHub Codespace**. The learner retains full control of git operations (stage, commit, push).
