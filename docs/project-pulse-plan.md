# Project Pulse implementation plan

## 1. Summary

Mona's team needs **Project Pulse**, a small static dashboard that shows contributors which projects are active, who owns each one, its current status, its recent activity, its priority or risk level, and a short summary. Each project appears as a card with a status badge.

The build produces four files:

| File | Owner | Purpose |
| --- | --- | --- |
| `app/index.html` | **Coder** | The page structure plus an inline script that loads `project-data.json` and renders the cards |
| `app/styles.css` | **Designer** | All visual design: layout, cards, badges, priority treatment, responsive and accessible styling |
| `app/project-data.json` | **Coder** | Sample data with a top-level `projects` array |
| `.vscode/launch.json` | **Coder** | The **Run Project Pulse Dashboard** launch configuration, which serves `app/` and opens `index.html` |

**How data is loaded:** `app/index.html` uses an inline `<script>` that calls `fetch("project-data.json")` with a relative URL. A local HTTP server, `python3 -m http.server 5500`, serves the page from `app/`. Opening `index.html` straight from disk (`file://`) is **not supported**, because browsers block or restrict `fetch` on `file://` URLs. In that case the page shows a clear error that tells the learner to use the launch configuration. No separate JS file, framework, build step, or npm dependency is added. This keeps the deliverable to exactly the three app files that the brief and the Step 3 checks expect.

**What this plan is based on:** `.github/project-pulse-brief.md`, `.github/agents/*.agent.md`, `docs/agent-team.md`, `.github/steps/2-step.md` to `4-step.md`, and `.github/workflows/2-step.yml` and `3-step.yml`. The Step 3 workflow checks for these literal strings and settings, so the plan keeps them exactly:

- the title "Project Pulse"
- `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority` in `index.html`
- `.dashboard`, `.project-card`, `border-radius` and `box-shadow` in `styles.css`
- a parseable `project-data.json` with `projects`, `name`, `owner`, `status`, `recentActivity` and `priority`
- a parseable `launch.json` that contains "Run Project Pulse Dashboard" and `index.html`

**Git:** No agent stages, commits, or pushes. The learner does all Git operations through Copilot CLI prompts.

---

## 2. File assignments

Each file has exactly one owner. An agent may *read* any file but may *edit* only the files it owns. Changes to a file owned by another agent go through the Orchestrator.

### `app/index.html` — owner: Coder

- Starts with `<!DOCTYPE html>` and has `<html lang="en">`, `<meta charset="utf-8">` and `<meta name="viewport" content="width=device-width, initial-scale=1">`.
- `<title>Project Pulse</title>` and a visible `<h1>` with the exact text **Project Pulse**.
- `<link rel="stylesheet" href="styles.css">`, using a relative path.
- `<link rel="icon" href="data:,">`. Without it, the browser requests `/favicon.ico`, gets a 404 from `http.server`, and logs a console error.
- Static page markup that follows the **HTML/CSS contract** in section 5.2. That includes the `.dashboard` root, the header, the message/live region, and the empty card list container.
- A `<noscript>` message saying JavaScript is needed to load project data.
- One inline `<script>` (no external JS file) that:
  1. Fetches `project-data.json` with a relative URL and `{ cache: "no-store" }`.
  2. Checks `response.ok` and throws a clear error that includes the HTTP status if it is false.
  3. Parses the JSON and checks that `data.projects` is an array.
  4. Normalizes every project field (see section 7), then builds one `.project-card` per project with `document.createElement` and `textContent`. It does **not** put data into the page with `innerHTML`.
  5. Shows the status, `recentActivity`, priority, owner and name in each card. The words `status`, `recentActivity` and `priority` will appear in the script source, which satisfies the Step 3 checks.
  6. Shows a loading state, an empty state and an error state in the message region.
  7. Calls `console.error` only on real failures. A successful load logs nothing.
- Keeps the order of projects in the JSON file, so the output is deterministic.

### `app/styles.css` — owner: Designer

- Design tokens as CSS custom properties on `:root`: colors, spacing scale, radius, shadow and font stack.
- A `.dashboard` selector for the page container: max width, centered, padding.
- A `.project-card` selector with `border-radius`, `box-shadow`, padding, background and a clear internal hierarchy.
- A responsive card grid on `.project-grid`, for example `repeat(auto-fill, minmax(18rem, 1fr))`, which drops to a single column on narrow screens.
- Status badges (`.status-badge` plus a modifier for each status, including `--unknown`).
- Priority treatment (`.priority-badge` plus modifiers, and a card-level accent such as a colored left border through `.project-card--priority-*`). The accent always comes with a text label, so color is never the only signal.
- Message states (`.dashboard-message`, `--loading`, `--empty`, `--error`).
- Accessibility basics:
  - WCAG AA contrast: 4.5:1 for body text, 3:1 for large text and badge borders.
  - Visible `:focus-visible` styles.
  - `@media (prefers-reduced-motion: reduce)` turns off any transitions.
  - Long text wraps (`overflow-wrap: anywhere`).
- No external fonts or CDNs, so the page works offline in Codespaces.

### `app/project-data.json` — owner: Coder

- Strict JSON with a top-level object `{ "projects": [ ... ] }`.
- 5–6 sample projects that together use every status value and every priority value in the schema (section 5.1), so all badge styles can be checked visually.
- Every project has `name`, `owner`, `status`, `recentActivity` and `priority`. Projects may also have `summary` (the brief asks for "a short contributor-friendly summary").
- Fictional or GitHub-mascot data only (for example Mona, Hubot, Octocat). No real personal data and no secrets.

### `.vscode/launch.json` — owner: Coder

Strict JSON with **no comments** and no trailing commas, so `python3 -m json.tool` accepts it. It contains `"version": "0.2.0"` and one configuration:

| Key | Value | Reason |
| --- | --- | --- |
| `name` | `Run Project Pulse Dashboard` | Exact name that Step 3 and Step 4 check for |
| `type` | `node-terminal` | Built into VS Code's JavaScript debugger, so no extra extension is needed. It runs a shell command in the integrated terminal, which is a TTY, so the server's startup line is not buffered |
| `request` | `launch` | Standard |
| `command` | `python3 -m http.server 5500` | Exact command from Step 3. Python 3 is already in the Codespace universal image |
| `cwd` | `${workspaceFolder}/app` | Serves `app/` as the web root |
| `serverReadyAction.pattern` | `Serving HTTP on .* port ([0-9]+)` | Matches the `http.server` startup line for both IPv4 and IPv6 (`::`). Uses `[0-9]` instead of `\d` to avoid backslash escaping in JSON |
| `serverReadyAction.uriFormat` | `http://localhost:%s/index.html` | Opens the dashboard itself, not a directory listing |
| `serverReadyAction.action` | `openExternally` | Opens the browser. Codespaces forwards the port automatically |

The existing `.vscode/tasks.json` must **not** be changed.

---

## 3. Designer responsibilities

1. Own and write `app/styles.css`. Do not edit any other file.
2. Style **only** the class names and elements in the contract (section 5.2). If a new hook is needed, ask the Orchestrator so the Coder can add it to `app/index.html`. Do not edit HTML directly.
3. Make the first view clearly read as the Project Pulse dashboard:
   - a strong title and subtitle
   - a card grid with rounded corners and soft shadows
   - clear spacing and typography hierarchy: project name, then status and priority, then owner and activity, then summary
4. Define distinct, accessible styles for every status value and every priority value, plus a neutral `--unknown` style for each.
5. Make the layout responsive at about 360px, 768px and 1280px or wider, with no horizontal scrolling at 320px.
6. Meet the accessibility basics listed in section 2 (contrast, focus, reduced motion, text wrapping, no color-only meaning).
7. Report design decisions, tradeoffs (for example, the chosen priority accent and why), the files touched, and how to check the result visually.

## 4. Coder responsibilities

1. Own and write `app/project-data.json`, `app/index.html` and `.vscode/launch.json`. Do not edit `app/styles.css`.
2. Produce markup and class names that match the contract (section 5.2) **exactly**, including modifier naming and how status and priority values are normalized.
3. Write the inline fetch-and-render script with explicit, informative errors and a deterministic output order.
4. Write the data file to the schema (section 5.1), covering every status and priority value.
5. Create `.vscode/launch.json` as strict JSON with the exact values in section 2.
6. Run the command-line checks in section 8 and report what changed, what was checked, and any remaining risk.

---

## 5. Shared contract between agents

This contract is the only thing that ties the Designer's work to the Coder's work. It is fixed by this plan, so both agents can start at the same time. Any change to it must go through the Orchestrator, and both agents then update their files in sequence.

### 5.1 Data schema (`app/project-data.json`)

| Field | Type | Required | Allowed values / notes |
| --- | --- | --- | --- |
| `projects` | array (top level) | yes | May be empty |
| `projects[].name` | string | yes | Shown as the card heading |
| `projects[].owner` | string | yes | A person or team name |
| `projects[].status` | string | yes | `active`, `at-risk`, `blocked`, `paused`, `completed` |
| `projects[].recentActivity` | string | yes | Short human-readable sentence, for example "Merged search filters PR · 2 days ago" |
| `projects[].priority` | string | yes | `high`, `medium`, `low` |
| `projects[].summary` | string | no | One or two sentences written for contributors |

**Display labels:**

- Status: `active` → "Active", `at-risk` → "At risk", `blocked` → "Blocked", `paused` → "Paused", `completed` → "Completed".
- Priority: `high` → "High", `medium` → "Medium", `low` → "Low".

**Normalization rule (Coder):** trim the value, lowercase it, and replace spaces and underscores with `-`. If the result is not in the allowed list, the modifier becomes `unknown` and the badge shows the raw trimmed text, or "Unknown" if the value is empty.

### 5.2 HTML structure and class names (what `app/styles.css` relies on)

```
body
└── main.dashboard                     (aria-labelledby="dashboard-title")
    ├── header.dashboard-header
    │   ├── h1#dashboard-title.dashboard-title     "Project Pulse"
    │   └── p.dashboard-subtitle
    ├── p.dashboard-message[role=status][aria-live=polite]
    │      + one modifier: .dashboard-message--loading | --empty | --error
    │      (hidden attribute when there is nothing to show)
    └── ul.project-grid                (list of cards)
        └── li.project-grid__item
            └── article.project-card.project-card--priority-{high|medium|low|unknown}
                   [data-status="{normalized}"] [data-priority="{normalized}"]
                   [aria-labelledby="project-{index}-name"]
                ├── header.project-card__header
                │   ├── h2#project-{index}-name.project-card__name
                │   └── span.status-badge.status-badge--{active|at-risk|blocked|paused|completed|unknown}
                ├── p.project-card__summary            (only if summary is present)
                └── dl.project-card__meta
                    ├── div.project-card__field  → dt "Owner"           / dd.project-card__owner
                    ├── div.project-card__field  → dt "Priority"        / dd > span.priority-badge.priority-badge--{high|medium|low|unknown}
                    └── div.project-card__field  → dt "Recent activity" / dd.project-card__activity
```

- `{index}` is the zero-based array index. Using the index keeps IDs unique even when two projects have the same name.
- The badge's visible text is the display label, so the badge never depends on color alone.
- The Designer must not depend on any element or class not listed here.

---

## 6. Ordered implementation steps and dependencies

| # | Step | Agent | Files | Depends on |
| --- | --- | --- | --- | --- |
| 0 | Approve this plan and freeze the contract (section 5) | Orchestrator / learner | `docs/project-pulse-plan.md` (already written) | — |
| 1a | Write the visual design | Designer | `app/styles.css` | 0 |
| 1b | Write the sample data to the schema | Coder | `app/project-data.json` | 0 |
| 1c | Write the page markup and render script to the contract | Coder | `app/index.html` | 0. Follows 1b inside the Coder's task so the data can be used to test the script |
| 1d | Write the launch configuration | Coder | `.vscode/launch.json` | 0 |
| 2 | Integration run: start **Run Project Pulse Dashboard** and check that the cards render with the styles applied | Coder runs it. Designer reviews the visuals | read-only | 1a, 1b, 1c, 1d |
| 3 | Fix-ups: CSS issues go to the Designer, and markup, script, data or launch issues go to the Coder. A contract change means the Coder updates first, then the Designer | Owning agent | the owning agent's files only | 2 |
| 4 | Final validation (section 8) and a summary to the learner | Orchestrator, with checks delegated | read-only | 3 |

**Dependencies in short:**

- Both agents depend only on the contract, not on each other's output.
- `app/index.html` depends on the schema in `app/project-data.json` at runtime.
- The look of `app/index.html` depends on `app/styles.css`.
- `.vscode/launch.json` depends only on the `app/` directory and the `index.html` filename.
- The integration run depends on all four files.

---

## 7. Parallel work decisions

**Can run in parallel (Phase 1):**

- **Designer → `app/styles.css`** and **Coder → `app/project-data.json`, `app/index.html`, `.vscode/launch.json`**.
  - The file scopes do not overlap: each file has exactly one owner.
  - There is no data dependency at authoring time: the CSS hooks and data schema are fixed in section 5, so neither agent waits for the other's output.

**Must run sequentially:**

- **Phase 0 comes before Phase 1.** Both agents need the frozen contract. Starting before it is approved risks class-name drift.
- **Inside the Coder's task:** `project-data.json`, then `index.html`. The render script is tested against real data. These are the same agent's files, so this is ordering, not a conflict. `launch.json` can be written at any point.
- **Phase 2 comes after all of Phase 1.** The integration run needs all four files.
- **Phase 3 fix-ups** are sequential whenever a fix touches the contract. The Coder changes the markup first, then the Designer updates the selectors. Fixes that stay inside one owner's file can be done in parallel.
- **Phase 4 comes last.** Validation must run against the final files.
- **No agent performs Git operations at any point.** The learner commits after reviewing.

---

## 8. Edge cases to handle

| Case | Expected behavior | Owner |
| --- | --- | --- |
| `projects` is an empty array | Message region shows `--empty`: "No projects to show yet." The grid is empty and nothing is logged to the console | Coder (logic), Designer (style) |
| `projects` missing or not an array | `--error`: "Project data is missing a projects list." Logged with `console.error` | Coder |
| Invalid JSON | `--error` that mentions the data could not be parsed. Logged with `console.error` | Coder |
| HTTP error (404, 500) | `--error` that includes the status code | Coder |
| Opened via `file://` (fetch fails with `TypeError`) | `--error`: "Could not load project data. Open the dashboard with the **Run Project Pulse Dashboard** launch configuration." | Coder |
| Missing or empty `name` | Show "Untitled project" | Coder |
| Missing `owner` | Show "Unassigned" | Coder |
| Missing `recentActivity` | Show "No recent activity" | Coder |
| Missing or unknown `status` | `status-badge--unknown`, label is the raw text or "Unknown" | Coder / Designer |
| Missing or unknown `priority` | `priority-badge--unknown` and `project-card--priority-unknown` | Coder / Designer |
| Non-string field values (number, null, object) | Strings and numbers are converted to text. Anything else falls back as above. Never render `[object Object]` | Coder |
| Mixed case or spacing, for example `"At Risk"` | Normalized to `at-risk` | Coder |
| Missing `summary` | Leave out the summary paragraph. No empty gap | Coder / Designer |
| Very long names, owners or activity text | Text wraps inside the card and does not overflow the layout | Designer |
| HTML-like text in data | Rendered as plain text through `textContent` | Coder |
| Duplicate project names | Unique IDs come from the array index | Coder |
| Many projects (20+) | The grid wraps naturally and does not break the layout | Designer |
| JavaScript disabled | `<noscript>` message is shown | Coder |
| Port 5500 already in use | `http.server` exits with "Address already in use" and no browser opens. Fix: stop the previous run or terminal, then launch again | Documented in the handoff |
| Codespaces port forwarding | `serverReadyAction` opens the forwarded URL. If a pop-up is blocked, use the Ports panel to open port 5500 and add `/index.html` | Documented in the handoff |
| Favicon 404 noise | Prevented by `<link rel="icon" href="data:,">` | Coder |

---

## 9. Validation expectations

### 9.1 Static file checks (Coder runs from the repo root)

1. `python3 -m json.tool app/project-data.json` succeeds.
2. `python3 -m json.tool .vscode/launch.json` succeeds. This proves the file is strict JSON with no comments or trailing commas.
3. A short Python one-liner confirms:
   - `projects` is a list with at least one entry
   - every entry has non-empty string values for `name`, `owner`, `status`, `recentActivity` and `priority`
   - every status and priority is in the allowed list
   - the data covers all five statuses and all three priorities
4. `launch.json` contains:
   - `"name": "Run Project Pulse Dashboard"`
   - `"cwd": "${workspaceFolder}/app"`
   - `"command": "python3 -m http.server 5500"`
   - `uriFormat` set to `http://localhost:%s/index.html`
5. Literal-string checks that mirror the Step 3 workflow:
   - `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
   - `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
6. `.vscode/tasks.json` is unchanged.

### 9.2 Served checks (Coder)

1. `cd app && python3 -m http.server 5500` (or the launch configuration) starts and prints `Serving HTTP on ... port 5500`.
2. `curl -s -o /dev/null -w "%{http_code}" http://localhost:5500/index.html` returns `200`, and the body contains "Project Pulse".
3. `curl -s http://localhost:5500/project-data.json` returns `200` and valid JSON.
4. Stop the server after the checks.

### 9.3 Browser checks (Designer reviews the visuals, Coder reviews behavior)

1. **Run Project Pulse Dashboard** opens `/index.html`, not a directory listing.
2. The page shows the "Project Pulse" heading inside `main.dashboard`, and one `article.project-card` per project. In DevTools, `document.querySelectorAll(".dashboard .project-card").length` equals the number of projects.
3. Every card shows the name, owner, a status badge, a priority badge or label, and recent activity. Each status and priority value looks different from the others.
4. Cards have rounded corners and shadows, and the spacing is readable.
5. **Responsive:** at 320–375px the cards are in one column with no horizontal scroll. At about 768px there are 2 columns, and at 1280px or wider there are 3 or more.
6. **Accessibility basics:**
   - exactly one `h1`, and one `h2` per card
   - `lang="en"` is set
   - text contrast is at least 4.5:1, as checked with DevTools or Lighthouse
   - focus is visible when tabbing through the page
   - status and priority are readable as text, not only as color
   - the message region is announced through `role="status"`
   - a Lighthouse accessibility score of 90 or higher is the target
7. **Console:** no errors or warnings on a successful load, and no 404s in the Network panel (including favicon).
8. **Edge-case checks** use temporary local edits that are reverted afterwards and never left in the final files:
   - an empty `projects` array shows the empty state
   - a misspelled data filename or broken JSON shows the error state
   - an unknown status shows the neutral badge
   - opening via `file://` shows the error with the launch hint

### 9.4 Do not use as a learner check

`scripts/validate-exercise.sh` is a template-maintenance script. It reports **FAIL** once the learner's files are committed (the "Learner answer files are not tracked" check), so it should not be used to judge Project Pulse.

---

## 10. Open questions

1. **Status vocabulary:** is `active / at-risk / blocked / paused / completed` the right set for Mona's team, or should it be `on-track / at-risk / off-track`? The plan uses the first set, and changing it only touches the contract labels and modifiers.
2. **Card order:** the plan keeps the JSON order. Should cards instead be sorted, for example by priority and then by status?
3. **Extra fields:** should the schema add `lastUpdated` (an ISO date) or a link to each project's repository? These are not in the brief, so they are left out for now.
4. **Summary counts:** would a small summary bar at the top (for example "3 active · 1 at risk · 1 blocked") be useful? It is left out to keep the scope minimal and could be a follow-up.
5. **Launch type:** `node-terminal` relies on VS Code's built-in JavaScript debugger, which is present in Codespaces by default. If the learner's environment doesn't have it, should the Coder switch to a `debugpy` configuration? That would need the Python extension, which is not in `devcontainer.json`.
6. **Who saves files the Orchestrator can't edit:** the Orchestrator's tools are `read`, `agent` and `memory`, with no `edit`. Saving this plan and the later `docs/final-handoff.md` needs a delegated agent or Copilot CLI's default agent. Which one should do it?
