# Project Pulse: final handoff

Project Pulse is a static dashboard for Mona's team. It shows each project's name, owner, status, recent activity and priority. A custom agent team built it from Copilot CLI in a Codespace.

## Agent team and what each agent did

| Agent | What it did |
| --- | --- |
| **Planner** | Wrote `docs/project-pulse-plan.md`. It covers the data and class-name contract, file assignments, edge cases and validation expectations. |
| **Orchestrator** | Split the plan into phases. Ran the Designer and Coder in parallel because their file scopes did not overlap. Did the integration review. Delegated the git operations (at the learner's request) and this handoff. |
| **Designer** | Owned `app/styles.css`. Built the indigo gradient header and white rounded cards with layered shadows. Each card has a 6px left border whose accent color shows priority. Status pills use a different marker shape for each status. Priority chips are outlined and include signal bars. The grid is responsive (`auto-fill`). The design targets AA contrast, shows `:focus-visible` outlines and supports `prefers-reduced-motion`. |
| **Coder** | Owned `app/index.html`, `app/project-data.json` and `.vscode/launch.json`. An inline script fetches the data and renders it with `createElement` and `textContent`. It normalizes status and priority values and shows loading, empty and error states. Added 6 sample projects and the launch configuration. Ran the validation below and wrote this handoff. |

## Deliverables

| File | Owner | Purpose |
| --- | --- | --- |
| `app/index.html` | Coder | Page structure plus an inline script that fetches and renders the project cards |
| `app/styles.css` | Designer | All visual styling, responsive grid and accessibility states |
| `app/project-data.json` | Coder | 6 sample projects covering all 5 statuses and all 3 priorities |
| `.vscode/launch.json` | Coder | The **Run Project Pulse Dashboard** launch configuration |

## How to run

1. Open the **Run and Debug** panel in VS Code (`Ctrl+Shift+D` / `Cmd+Shift+D`).
2. Select **Run Project Pulse Dashboard** from the drop-down.
3. Press **Start Debugging** (the green ▶, or `F5`).
4. This starts `python3 -m http.server 5500` in `app/`. When the server is ready, the dashboard opens at http://localhost:5500/index.html.

## Dashboard validation results

The Coder ran these checks on 2026-11-15 against commit `269220a`. All of them passed.

| # | Check | Result |
| --- | --- | --- |
| 1 | `python3 -m json.tool app/project-data.json` | ✅ Pass |
| 2 | `python3 -m json.tool .vscode/launch.json` (strict JSON) | ✅ Pass |
| 3 | Data schema: `projects` is a non-empty list, and every entry has non-empty string `name`, `owner`, `status`, `recentActivity` and `priority` | ✅ Pass (6 projects) |
| 4 | Every status is in the allowed list, and all five are covered: active, at-risk, blocked, paused, completed | ✅ Pass |
| 5 | Every priority is in the allowed list, and all three are covered: high, medium, low | ✅ Pass |
| 6 | `launch.json` values: name, `type: node-terminal`, `command`, `cwd: ${workspaceFolder}/app` and `uriFormat: http://localhost:%s/index.html` | ✅ Pass |
| 7 | The `serverReadyAction` pattern `Serving HTTP on .* port ([0-9]+)` matches the real startup line `Serving HTTP on 0.0.0.0 port 5500 (http://0.0.0.0:5500/) ...` | ✅ Pass. Captures `5500` with both the Python and the JS regex engines |
| 8 | `index.html` literal strings: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` and `<title>Project Pulse</title>` | ✅ Pass |
| 9 | `styles.css` literal strings: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`. Responsive layout uses `repeat(auto-fill, …)` plus `@media` queries | ✅ Pass |
| 10 | Class contract: all 33 class names that `index.html` generates (grid item, card elements, priority/status/badge modifiers including `--unknown`, message states, header classes) have a matching selector in `styles.css` | ✅ Pass (0 missing) |
| 11 | Served: `GET /index.html` returns 200 and the body contains "Project Pulse" | ✅ Pass |
| 12 | Served: `GET /project-data.json` returns 200 (`application/json`) and valid JSON. `GET /styles.css` returns 200 | ✅ Pass |
| 13 | After the checks, the server was stopped and port 5500 was free | ✅ Pass |
| 14 | The inline script was run in Node v22 against a minimal fake DOM with the real data and rendered 6 `.project-card` elements with correct badges. The loading message was hidden afterwards | ✅ Pass |
| 15 | Edge cases in the fake DOM: an empty list shows the empty state. Broken JSON, HTTP 404, a network/`file://` failure and a missing `projects` key each show the matching error message. An unknown status or priority gets the `--unknown` badge. `"At Risk"` and `"HIGH"` are normalized. HTML in the data is rendered as plain text | ✅ Pass |
| 16 | `.vscode/tasks.json` is unchanged | ✅ Pass |
| 17 | `git status --short` shows only this handoff file as new | ✅ Pass |

The edge-case tests used stubbed responses in `/tmp`. No repository file was edited for testing.

## Learner handoff notes

### How to run

Use **Run and Debug → Run Project Pulse Dashboard → Start**. The dashboard opens at http://localhost:5500/index.html. To run it by hand instead, use `cd app && python3 -m http.server 5500`.

### Known limitations

- **No real-browser check:** the agents did not do a real-browser or Lighthouse check. The visuals, responsive breakpoints, contrast, focus order and console cleanliness (plan section 9.3) still need a human check in a browser.
- **Debugger dependency:** the `node-terminal` launch type needs VS Code's built-in JavaScript debugger. It is included in Codespaces by default.
- **Port 5500 in use:** if the port is busy, `http.server` exits with "Address already in use" and no browser opens. Stop the previous run or terminal, then launch again.
- **Blocked pop-up:** if the browser blocks the pop-up (common in Codespaces), open port 5500 from the **Ports** panel and add `/index.html` to the URL.
- **`file://` not supported:** opening `index.html` directly from disk shows an error that tells you to use the launch configuration. Browsers block `fetch` from `file://` pages.
- `scripts/validate-exercise.sh` is a template-maintenance script. Don't use it to judge Project Pulse (see plan section 9.4).

### Open questions (from plan section 10, still open)

1. **Status vocabulary:** keep `active / at-risk / blocked / paused / completed`, or switch to `on-track / at-risk / off-track`?
2. **Card order:** keep the JSON order, or sort by priority and then status?
3. **Extra fields:** add `lastUpdated` (ISO date) or a repository link per project?
4. **Summary counts:** add a summary bar such as "3 active · 1 at risk · 1 blocked"?
5. **Launch type:** if VS Code's JS debugger isn't available, switch to a `debugpy` configuration? That would need the Python extension.
6. **File-saving ownership:** the Orchestrator has no `edit` tool. Which agent should own saving docs like this one? This time the Coder did it.

## Git status

- The dashboard files were committed as `269220a` "Build the Project Pulse dashboard" and pushed to `main` at the learner's request.
- This handoff file (`docs/final-handoff.md`) is **not yet committed**. Staging, committing and pushing are up to the learner.
