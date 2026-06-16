# Project Pulse — Implementation Plan

## Summary

Project Pulse is a browser-based dashboard that displays the health and priority of active projects. The implementation is split across four phases: a data layer, a styling layer, the HTML structure with data wiring, and a launch configuration. Phases 1 and 2 have no file overlap and can run in parallel. Phases 3 and 4 must run sequentially after the earlier phases complete.

## Implementation steps

### Phase 1 — Data layer (Coder)

- **Goal:** Create the sample project dataset that drives all card rendering.
- **Specialist:** Coder
- **Files:** `app/project-data.json`
- **Depends on:** none
- **Can run in parallel with:** Phase 2

**Steps:**
1. Create the `app/` directory.
2. Create `app/project-data.json` as a JSON array of project objects.
3. Include at least 4 projects covering every combination of status (`On Track`, `At Risk`, `Blocked`) and priority (`High`, `Medium`, `Low`).
4. Each object must have: `id`, `name`, `status`, `priority`, `description`.
5. Validate the file parses as valid JSON.

**Edge cases:**
- Extra fields in the JSON should not break rendering (HTML ignores unknown keys).
- Empty array would produce a blank dashboard — ensure at least 4 entries.

**Validation:**
- `node -e "JSON.parse(require('fs').readFileSync('app/project-data.json','utf8'))"` exits 0.
- File contains entries for all three status values and all three priority values.

---

### Phase 2 — Dashboard styling (Designer)

- **Goal:** Produce a polished CSS stylesheet that makes the dashboard look like a real product UI.
- **Specialist:** Designer
- **Files:** `app/styles.css`
- **Depends on:** none
- **Can run in parallel with:** Phase 1

**Steps:**
1. Create `app/styles.css`.
2. Define a `.dashboard` container with responsive grid layout (`auto-fill`, `minmax(280px, 1fr)`).
3. Define `.project-card` with rounded corners, box shadow, padding, and clear typography.
4. Define status badge classes: `.status-on-track` (green), `.status-at-risk` (amber), `.status-blocked` (red).
5. Define priority classes: `.priority-high`, `.priority-medium`, `.priority-low` with distinct visual treatments.
6. Add a page header style (`h1` or `.dashboard-header`).
7. Ensure sufficient color contrast for accessibility (WCAG AA minimum).
8. Add basic responsive behavior so cards reflow on narrow viewports.

**Edge cases:**
- Long project names or descriptions should not break card layout — use `overflow-wrap: break-word`.
- Cards with no priority or status class should degrade gracefully.

**Validation:**
- Open `app/index.html` in a browser and confirm cards are visually distinct and polished.
- Resize the viewport to < 600 px and confirm cards reflow to a single column.
- Run a contrast checker on badge text/background pairs.

---

### Phase 3 — HTML structure and data wiring (Coder)

- **Goal:** Build the HTML shell and JavaScript that fetches `project-data.json` and renders project cards into the page.
- **Specialist:** Coder
- **Files:** `app/index.html`
- **Depends on:** Phase 1, Phase 2
- **Can run in parallel with:** none

**Steps:**
1. Create `app/index.html` with valid HTML5 boilerplate.
2. Link `styles.css` in `<head>`.
3. Add a `<h1>` page title ("Project Pulse") and a `<div class="dashboard">` container.
4. Add an inline `<script>` (or external `app.js`) that calls `fetch('project-data.json')`.
5. On success, iterate the project array and for each project insert a `.project-card` element containing:
   - Project name in a heading (`<h2>`)
   - Status badge `<span>` with class `status-<normalised-status>` (spaces → hyphens, lowercase)
   - Priority indicator with class `priority-<lowercase-priority>`
   - Description paragraph
6. On fetch error, display a user-visible error message inside `.dashboard`.
7. Validate the page renders with no console errors when served from a local HTTP server.

**Edge cases:**
- `fetch()` fails if `index.html` is opened as a `file://` URL — the launch config (Phase 4) must serve via HTTP.
- Status and priority values must be normalised to safe CSS class names (lowercase, spaces to hyphens).
- Missing fields in a project object should not throw — use fallback strings.

**Validation:**
- Open via Live Server and confirm all 4+ project cards appear.
- Browser console shows no errors.
- Each card displays name, status badge, priority, and description.

---

### Phase 4 — Launch configuration (Coder)

- **Goal:** Create a VS Code launch configuration so the learner can open the dashboard with one click.
- **Specialist:** Coder
- **Files:** `.vscode/launch.json`
- **Depends on:** Phase 3
- **Can run in parallel with:** none

**Steps:**
1. Create `.vscode/` directory if it does not exist.
2. Create `.vscode/launch.json` as strict JSON (no comments).
3. Add a configuration named `"Open Project Pulse"` using the Live Server extension (`"type": "vscode-edge-devtools.debug"` or a simple browser launch that opens `index.html`).
4. Set `"cwd"` to `"${workspaceFolder}/app"`.
5. Set the URL to `"http://localhost:5500/index.html"` (Live Server default port).
6. Validate the JSON is well-formed.

**Edge cases:**
- If Live Server is not installed the config will fail silently — note this in the dashboard README or open questions.
- Port 5500 may be in use — document the fallback (change port in Live Server settings).

**Validation:**
- `.vscode/launch.json` parses as valid JSON.
- Opening the launch config in VS Code shows the named configuration in the Run panel.
- Launching opens `app/index.html` in the browser via HTTP (not `file://`).

---

## Parallel work decisions

| Phases | Can run in parallel? | Reason |
|---|---|---|
| Phase 1 + Phase 2 | ✅ Yes | No shared files. Data and styles are independent artifacts. |
| Phase 2 + Phase 3 | ❌ No | Phase 3 consumes `styles.css`; Designer must finish before Coder wires the HTML. |
| Phase 1 + Phase 3 | ❌ No | Phase 3 depends on the JSON schema established in Phase 1. |
| Phase 3 + Phase 4 | ❌ No | Launch config must reference a working `index.html`. |

---

## Sequential constraints

1. **Phase 3 requires Phase 1 to be complete** — the HTML/JS must know the JSON schema (field names) before it can render cards.
2. **Phase 3 requires Phase 2 to be complete** — CSS class names agreed in `styles.css` must exist before the HTML uses them, or cards will render without styles.
3. **Phase 4 requires Phase 3 to be complete** — the launch config opens `index.html`; that file must exist and work.

---

## Designer responsibilities

- Own `app/styles.css` end-to-end.
- Define all visual design tokens (colours, spacing, typography, shadow, radius).
- Implement the `.dashboard` grid layout and `.project-card` card component.
- Define status badge classes (`.status-on-track`, `.status-at-risk`, `.status-blocked`).
- Define priority classes (`.priority-high`, `.priority-medium`, `.priority-low`).
- Ensure WCAG AA colour contrast on all text/background pairs.
- Implement responsive breakpoints so the grid reflows on narrow viewports.
- Use `overflow-wrap: break-word` to protect layout from long strings.
- Coordinate CSS hook names (`.dashboard`, `.project-card`) with the Coder before Phase 3 begins.

---

## Coder responsibilities

- Own `app/project-data.json`, `app/index.html`, and `.vscode/launch.json`.
- Define the project data schema: `id`, `name`, `status`, `priority`, `description`.
- Populate at least 4 sample projects covering all status and priority enum values.
- Write the HTML5 shell with correct `<head>` links and semantic structure.
- Implement `fetch('project-data.json')` and dynamic card rendering in JavaScript.
- Normalise status/priority strings to safe CSS class names before applying them.
- Handle fetch errors with a user-visible fallback message.
- Create `.vscode/launch.json` as strict JSON with `cwd: ${workspaceFolder}/app`.
- Validate that the app works when served via HTTP (not `file://`).

---

## Validation expectations

End-to-end checklist to confirm the dashboard is complete and working:

- [ ] `app/project-data.json` is valid JSON and contains ≥ 4 projects.
- [ ] All three status values (`On Track`, `At Risk`, `Blocked`) appear in the data.
- [ ] All three priority values (`High`, `Medium`, `Low`) appear in the data.
- [ ] `app/index.html` opens in a browser served via HTTP without errors.
- [ ] Browser console shows zero errors or warnings on load.
- [ ] All project cards are visible with name, status badge, priority indicator, and description.
- [ ] Status badges are visually distinct (colour-coded).
- [ ] Priority indicators are visually distinct.
- [ ] Page heading "Project Pulse" is visible.
- [ ] Layout is responsive: cards reflow to single column at ≤ 600 px viewport width.
- [ ] `.vscode/launch.json` is valid JSON and appears in the VS Code Run panel.
- [ ] Launching from VS Code opens `http://localhost:5500/index.html` (or equivalent).

---

## Open questions

1. **Live Server dependency** — should the Coder add a note or README reminding the learner to install the VS Code Live Server extension, or is it pre-installed in the Codespace?
2. **External vs inline JS** — should the JavaScript be inline in `index.html` or in a separate `app/app.js`? Inline is simpler; a separate file is easier to test.
3. **CSS hooks agreement** — Coder and Designer must agree on the exact class names before Phase 3 starts. The plan uses `.dashboard`, `.project-card`, `.status-*`, and `.priority-*`. Confirm these before parallelising phases 1 and 2.
4. **Port conflicts** — if port 5500 is in use in the Codespace, what is the fallback port and URL for the launch config?
5. **Project count** — 4 projects is the minimum. Should the Coder add more for a richer demo, or keep it minimal to reduce noise?
