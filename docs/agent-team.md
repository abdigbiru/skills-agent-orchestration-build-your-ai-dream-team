# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate this custom agent team to build Mona's Project Pulse dashboard.

| Agent | Target model | Responsibility | Agent definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Coordinates the specialists without implementing code itself. Turns the Planner's plan into phases, assigns explicit file scopes, manages dependencies, and verifies the integrated dashboard. | [`.github/agents/orchestrator.agent.md`](../.github/agents/orchestrator.agent.md) |
| Planner | Claude Opus 5.5 (copilot) | Researches the repository, documentation, dependencies, and edge cases. Produces an ordered implementation plan with file assignments, validation expectations, and parallel or sequential work; does not write code. | [`.github/agents/planner.agent.md`](../.github/agents/planner.agent.md) |
| Coder | GPT-5.5 (copilot) | Implements dashboard code and logic, fixes bugs, and validates changes within assigned files. When assigned, adds runnable-app support such as `.vscode/launch.json` to open the dashboard's `index.html` from `app/`. | [`.github/agents/coder.agent.md`](../.github/agents/coder.agent.md) |
| Designer | Gemini 3.7 Flash (copilot) | Shapes the dashboard's UI/UX, accessibility, information hierarchy, and responsive styling. Creates polished project cards, status badges, priority treatment, and consistent visual design within assigned files. | [`.github/agents/designer.agent.md`](../.github/agents/designer.agent.md) |

The Orchestrator starts with the Planner, then coordinates Coder and Designer tasks in parallel only when their file scopes do not overlap and no data dependencies exist. Overlapping or dependent tasks run sequentially. The learner controls staging, committing, and pushing through Copilot CLI prompts; the agents do not perform these Git operations themselves.
