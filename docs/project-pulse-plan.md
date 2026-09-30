# Project Pulse Dashboard — Implementation Plan

## Summary

Mona's team needs a small static frontend dashboard (`app/index.html`, `app/styles.css`, `app/project-data.json`) plus a VS Code launch configuration (`.vscode/launch.json`) that serves the app and opens the dashboard UI directly — not a directory listing. The dashboard must display multiple project cards with `name`, `owner`, `status`, `recentActivity`, and `priority`, styled with deterministic CSS hooks (`.dashboard`, `.project-card`) and polished visual treatment (`border-radius`, `box-shadow`, responsive layout). Work splits cleanly between **Designer** (visual system in `styles.css` + markup guidance) and **Coder** (HTML structure/logic, JSON data, launch config). The repo's own validation scripts (`scripts/validate-exercise.sh` and `.github/workflows/3-step.yml`) encode exact keyphrases and JSON-parseability checks that must be satisfied literally — this plan treats those as hard requirements, not suggestions.

Because `app/` does not yet exist, the first file created in it establishes the directory. No existing repo patterns for a frontend app exist to mirror (this is a template repo focused on the exercise/orchestration scaffolding), so this plan defines the conventions from the brief and agent definitions in `.github/agents/*.agent.md` and `.github/project-pulse-brief.md`.

## Exact schema for `app/project-data.json`

Top-level shape (required by validators: top-level `"projects"` key, and each project must literally contain the keys `name`, `owner`, `status`, `recentActivity`, `priority`):

```json
{
  "projects": [
    {
      "id": "atlas-onboarding",
      "name": "Atlas Onboarding Revamp",
      "owner": "Priya Shah",
      "status": "On Track",
      "priority": "High",
      "recentActivity": "Merged new onboarding flow into staging on Oct 2.",
      "progress": 72,
      "dueDate": "2025-11-15",
      "summary": "Redesigning first-run experience for new contributors."
    }
  ]
}
```

Notes:
- `status` should use a small closed set of values, e.g. `"On Track"`, `"At Risk"`, `"Blocked"`, `"Complete"` — Coder should pick 3–4 canonical strings and Designer should style badges keyed off them (see CSS hooks below).
- `priority` should use a small closed set, e.g. `"High"`, `"Medium"`, `"Low"` (or `"Critical"/"High"/"Medium"/"Low"`) — again a closed vocabulary so CSS can target it deterministically.
- `progress`, `dueDate`, `summary`, `id` are optional/realistic extras the brief allows ("a short contributor-friendly summary") but are not validator-checked — safe to include for polish, but must not replace the five required fields.
- Include **at least 4–6 realistic sample projects** so the grid/responsive layout is visually meaningful (a single card won't demonstrate "responsive layout").
- File must be strict, parseable JSON (`python3 -m json.tool app/project-data.json` is run in CI) — no comments, no trailing commas.

## Exact requirements for `app/index.html`

- Must contain the literal (case-insensitive) string **"Project Pulse"** somewhere visible (e.g., an `<h1>` title) — brief also asks the Designer/Coder prompt to use the *exact* title "Project Pulse".
- Must reference `styles.css` (literal substring `styles.css`, e.g. `<link rel="stylesheet" href="styles.css">`).
- Must reference `project-data.json` (literal substring, e.g. in a `fetch('project-data.json')` call or a `<script>` src, or a data-loading comment/attribute — the checker is a simple keyphrase grep against the raw HTML text so any literal occurrence of `project-data.json` satisfies it, but the real requirement is functional: fetch/render the JSON).
- Must contain a root/wrapper element carrying the class `dashboard` (Designer requires `.dashboard` CSS hook exist in styles.css; index.html must actually apply that class to some element for it to have effect, even though the CI check for `.dashboard` only checks `styles.css`, not html — still required for genuine rendering).
- Must render each project inside an element with class `project-card` (validator does a literal grep for `project-card` in `index.html`, and separately expects `.project-card` in `styles.css`).
- Must literally render (as visible DOM text, not just JSON keys) the values of `status`, `recentActivity`, and `priority` for each project — the CI keyphrase checker just does substring search on the raw file, so having the literal words `status`, `recentActivity`, `priority` appear anywhere in the HTML (e.g., as `data-*` attributes, labels, or template strings echoed into markup before JS executes) is what's mechanically checked. **However**, since this is a client-rendered app (data fetched via JS and injected into the DOM at runtime), the raw `index.html` source will NOT contain the actual status/activity/priority text unless:
  - the JS template literals in an inline `<script>` block include the literal field names (e.g. ``card.innerHTML = `<span class="status">${p.status}</span>`;`` — this itself contains the word `status` in the class name/markup, satisfying the grep), and/or
  - labels like `<span class="label">Status:</span>` are present in the JS template strings within `index.html`.
  
  **Edge case/risk:** If Coder writes an external JS file (e.g. `app/app.js`) instead of inline `<script>` in `index.html`, the keyphrase checks against `app/index.html` for `project-card`, `status`, `recentActivity`, `priority` will likely **fail** because those words would only exist in the external JS file, which is outside the checked/assigned file scope (only `app/index.html`, `app/styles.css`, `app/project-data.json`, `.vscode/launch.json` are in scope — no `app/app.js` is listed). **Recommendation: Coder must keep all JS inline inside `<script>` tags within `index.html`** (a `<script>` block at the end of body) rather than creating a separate JS file, both to stay within the assigned file scope and to satisfy the literal keyphrase checks reliably.
- Accessible markup: use semantic elements (`<main>`, `<h1>`, `<ul>`/`<section>` per card or `<article class="project-card">`), `alt`/`aria-label` where relevant, sufficient color contrast (Designer's job in CSS).

## Exact requirements for `app/styles.css`

- Must contain a `.dashboard` selector (literal).
- Must contain a `.project-card` selector (literal).
- Must contain `border-radius` (literal, anywhere — typically on `.project-card`).
- Must contain `box-shadow` (literal, anywhere — typically on `.project-card`).
- Should also implement: status badges (e.g. `.status-badge`, `.status-on-track`, `.status-at-risk`, `.status-blocked` or a generic `.badge` + `data-status` attribute selector), priority treatment (e.g. `.priority-high` with a distinct accent color/left-border), responsive grid (CSS Grid/Flexbox with `@media` breakpoints or `auto-fit`/`minmax`), spacing scale, typography (font sizing hierarchy for title vs. card body), and color contrast sufficient for readability (WCAG AA-ish, not mechanically checked but part of Designer's brief).
- No CI check requires `.status` or `.priority` class names specifically in CSS, but Designer's brief text explicitly calls for "status badges" and "clear priority treatment," so these are functional requirements even though not keyphrase-enforced.

## Exact requirements for `.vscode/launch.json`

- Must be **strict JSON, no comments, no trailing commas** (CI runs `python3 -m json.tool .vscode/launch.json`).
- Must contain the literal string `Run Project Pulse Dashboard` (a launch config `"name"`).
- Must contain the literal string `index.html` (used in the target URL).
- Functional requirements from the brief/step text (not all individually keyphrase-checked, but required for the exercise's manual "Run and Debug" verification in step 3):
  - `"cwd"` (or equivalent working-directory setting) set to `${workspaceFolder}/app`.
  - Serve using `python3 -m http.server 5500` (the step prompt explicitly specifies this command and port 5500).
  - Use a `serverReadyAction` (VS Code Node debug feature) with a `"uriFormat"` like `"http://localhost:%s/index.html"` and `"pattern"` matching the server's stdout to detect the port/ready state.

Since `launch.json` `serverReadyAction` is normally paired with a `"type": "node"`/`"pwa-node"` debug configuration launching a Node process, but the command required here is `python3 -m http.server 5500` (not Node), the standard-clean way to do this in VS Code is:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Project Pulse Dashboard",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceFolder}/app",
      "runtimeExecutable": "python3",
      "runtimeArgs": ["-m", "http.server", "5500"],
      "serverReadyAction": {
        "pattern": "Serving HTTP on .* port ([0-9]+)",
        "uriFormat": "http://localhost:%s/index.html",
        "action": "openExternally"
      }
    }
  ]
}
```

**Risk/edge case:** VS Code's `serverReadyAction` requires a debug session of certain types (`node`, `python`, `go`, etc., depending on extension support) to parse the console output; using `"type": "node"` with `runtimeExecutable: python3` is a known community pattern for wrapping arbitrary shell servers, but it is not officially documented as first-class for `python3 -m http.server` output pattern matching in all VS Code versions. The Coder should test this manually in the Codespace (Run and Debug → **Run Project Pulse Dashboard** → confirm browser opens `http://localhost:5500/index.html` showing the dashboard, not a directory listing) per step 3, item 6. If `serverReadyAction` does not reliably trigger with `type: node` + `runtimeExecutable`, an alternative is `"type": "python"` with `"module": "http.server"` and `args: ["5500"]`, which is more natively supported for Python-based servers and should be tried if the node-wrapper approach fails in the Codespace's actual VS Code/Python extension versions. Coder should validate this empirically rather than assume.

## Ordered Implementation Steps

### Phase 1 — Data contract (sequential, blocks everything else)
**Step 1: Define and create `app/project-data.json`**
- Owner: **Coder**
- File: `app/project-data.json`
- Action: Create the JSON file per the schema above with a top-level `"projects"` array and 4–6 realistic sample entries, each containing at minimum `name`, `owner`, `status`, `recentActivity`, `priority` (plus optional `progress`, `dueDate`, `summary`, `id`).
- Validate: `python3 -m json.tool app/project-data.json` succeeds; confirm literal substrings `projects`, `name`, `owner`, `status`, `recentActivity`, `priority` all appear.
- Why first: both `index.html` (fetch/render logic + literal field names in template strings) and `styles.css` (status/priority-driven selectors) are easier to write correctly once the exact field names and value vocabularies (e.g. status enum: On Track/At Risk/Blocked/Complete) are fixed. This is the single source of truth the other two files key off of.

### Phase 2 — Markup skeleton + visual system (can run in parallel once data contract is fixed)
**Step 2a: Draft `app/index.html` structure** — Owner: **Coder**
- Create the HTML skeleton: `<!DOCTYPE html>`, `<head>` with `<link rel="stylesheet" href="styles.css">`, `<title>Project Pulse</title>` (or similar), `<body>` with `<h1>Project Pulse</h1>` inside a container `<div class="dashboard">…</div>`, an empty `<section id="project-list"></section>` placeholder, and an inline `<script>` block (at end of body) that will `fetch('project-data.json')`, parse the `projects` array, and render one `<article class="project-card">` per project with visible text for `name`, `owner`, `status`, `recentActivity`, `priority` (all as template-literal-embedded field names + labels, e.g. `<span class="status status-${slugify(p.status)}">${p.status}</span>`).
- Coordinate with Designer on exact class names/hooks needed (see Step 2b) before finalizing markup — this is the one dependency between 2a and 2b; recommend a brief sync (Designer proposes class name conventions; Coder implements markup using them) rather than fully independent parallel work.

**Step 2b: Draft `app/styles.css` visual system** — Owner: **Designer**
- Implement `.dashboard` (page container: max-width, centered, padding, background), `.project-card` (border-radius, box-shadow, padding, background, transition/hover), status badge classes (e.g. `.status-badge`, modifiers per status value), priority treatment (e.g. left-border accent color or a `.priority-high/.priority-medium/.priority-low` class, or badge), typography scale (`h1`, card title, body text), spacing (consistent margin/gap via CSS Grid `gap`), and a responsive grid (`display:grid; grid-template-columns: repeat(auto-fit, minmax(280px,1fr)); gap:1.5rem;` with a `@media (max-width: 600px)` fallback to single column).
- Designer should specify (in report back to Orchestrator) the exact class-name contract it expects Coder's markup to use, since Designer only owns `styles.css` but its selectors are meaningless without matching HTML classes — this is the real "dependency" requiring a short synchronous handshake between Coder and Designer, even though the *files* they edit don't overlap.

**Parallelism note:** Because `index.html` and `styles.css` are different files owned by different agents, the Orchestrator *can* run Steps 2a and 2b in parallel — but only after they agree on a shared class-name contract (`.dashboard`, `.project-card`, status/priority class or attribute naming). Practically: have Designer propose the class contract first (fast), then run Coder's HTML implementation and Designer's CSS implementation in parallel against that agreed contract, or have Coder stub the markup with the required classes first and let Designer style against the existing DOM — either sequencing works as long as the class names are agreed before both finalize their files, since both files are graded independently for their own literal keyphrases (`.dashboard`/`.project-card` in CSS; `project-card`/`status`/`recentActivity`/`priority` in HTML) so no file-level merge conflict exists, only a semantic-contract dependency.

### Phase 3 — Launch configuration (sequential after Phase 2, or parallel if scope is well understood upfront)
**Step 3: Create `.vscode/launch.json`** — Owner: **Coder**
- File: `.vscode/launch.json`
- Depends on knowing the final path `app/index.html` exists (trivial, since path is fixed by the brief) — not truly blocked by Phase 2 completion, so this **can run in parallel** with Phase 2 since it doesn't touch `app/` files at all and has no data dependency on the HTML/CSS content, only on the fixed directory structure (`app/index.html` must exist as a filename, not any particular content).
- Create the JSON exactly as specified above: name `Run Project Pulse Dashboard`, `cwd` = `${workspaceFolder}/app`, command `python3 -m http.server 5500`, `serverReadyAction` opening `http://localhost:%s/index.html`.
- Validate: `python3 -m json.tool .vscode/launch.json` parses cleanly; manually run via VS Code Run and Debug to confirm browser opens the dashboard, not a directory listing.

### Phase 4 — Integration validation (sequential, after all files exist)
**Step 4: Cross-file validation** — Owner: **Coder** (or Orchestrator coordinating a final check)
- Confirm `index.html`'s fetch path (`project-data.json`) resolves correctly relative to `cwd=app` when served by the launch config (i.e., `fetch('project-data.json')` or `fetch('./project-data.json')`, not an absolute path that would break when served from `app/` as web root).
- Confirm all validator keyphrases pass by re-reading each file against the checklist in `.github/workflows/3-step.yml` / `scripts/validate-exercise.sh` (see below).
- Confirm visually (manual Run and Debug) that the dashboard shows cards, not a directory listing, and that responsive layout works by resizing the browser.

## File Assignments

| File | Owner | Notes |
|---|---|---|
| `app/project-data.json` | Coder | Created first; fixed field vocabulary drives HTML/CSS |
| `app/index.html` | Coder | Inline `<script>` only — no external JS file (stay in scope) |
| `app/styles.css` | Designer | Must match class-name contract agreed with Coder |
| `.vscode/launch.json` | Coder | Per `.github/agents/coder.agent.md`, launch config is Coder's responsibility |

No file is jointly edited by both agents — clean separation. The only cross-agent dependency is the **semantic class-name contract** between `index.html` markup and `styles.css` selectors, and the **field-name contract** from `project-data.json` consumed by `index.html`.

## Dependencies Between Steps

1. `app/project-data.json` → must exist (or at least have its schema fixed) before `app/index.html`'s render logic and `app/styles.css`'s status/priority-driven styles are finalized.
2. Class-name contract (`.dashboard`, `.project-card`, status/priority hooks) → must be agreed before both `index.html` and `styles.css` are considered "done," though both can be drafted in parallel against the agreed contract.
3. `.vscode/launch.json` → only depends on the fixed path `app/index.html` (by name), not on its content, so it has no real content-dependency and can be authored anytime, but should be **validated last** (after `index.html`/`styles.css` exist) since the manual "Run and Debug" smoke test needs the full app present to be meaningful.
4. Final integration validation → depends on all four files existing.

## Work That Can Run in Parallel

- `app/index.html` (Coder) and `app/styles.css` (Designer) — different files, no merge conflict, only a semantic contract dependency that can be resolved via a short upfront handshake.
- `.vscode/launch.json` (Coder) — can be authored concurrently with the above since it doesn't touch `app/` content, only references the fixed path.

## Work That Must Run Sequentially

- `app/project-data.json` must be substantially finalized (field names + status/priority vocab) **before** `index.html`'s render script and `styles.css`'s status/priority classes are finalized, to avoid rework.
- Final integration/manual smoke test (Run and Debug) must happen **after** all four files exist.
- If the Orchestrator assigns Coder to *both* `index.html`/`project-data.json` and `launch.json` (per this plan), that agent's own work is inherently sequential per its own task list even though there's no file conflict with Designer.

## Edge Cases to Handle

- **Fetch failing under `file://` protocol:** If a learner opens `index.html` directly via double-click (not through the launch config's HTTP server), `fetch('project-data.json')` will fail due to CORS/file:// restrictions in most browsers. Coder should add a visible error state (e.g., "Unable to load project data — make sure you're running the dashboard through the local server") rather than a silent blank page, and the brief's launch config exists precisely to avoid this failure mode — document this reliance in code comments.
- **Empty or malformed `projects` array:** Render logic should guard against `data.projects` being undefined/empty and show a friendly empty state rather than throwing.
- **Status/priority values with spaces or mixed case** (e.g., `"On Track"`) used in CSS class names: Coder must slugify (`"On Track"` → `on-track`) before interpolating into a `class="status-${slug}"` attribute, since raw spaces in class attributes break selectors.
- **JSON strictness:** No trailing commas, no comments, no single quotes in `project-data.json` or `launch.json` — both are parsed with `python3 -m json.tool` in CI; a syntax error fails the whole step.
- **External JS file temptation:** Keep all rendering logic inline in `index.html` per the file-scope constraint; an external `app/app.js` would functionally work in a browser but fails the CI's literal-keyphrase checks against `app/index.html` (since `project-card`, `status`, `recentActivity`, `priority` need to appear in that specific file) and is outside the declared file scope for this feature.
- **`serverReadyAction` compatibility:** As discussed above, using a non-Node command (`python3 -m http.server`) with VS Code's `serverReadyAction` is a known but slightly nonstandard pattern; must be manually tested in the actual Codespace/VS Code version, with a Python-debug-type fallback if the Node-wrapper approach doesn't trigger the port pattern reliably.
- **Port collisions:** Port `5500` is specified by the brief/step prompt exactly; if something else is already bound to 5500 in the Codespace, the server will fail to start — this is expected/deterministic per requirements, not something to change, but the Coder should note it as a known constraint.
- **Responsive layout verification:** "Responsive" must be visually confirmed by resizing the viewport/browser — not something the CI checks mechanically, so Designer/Coder should manually verify at common breakpoints (mobile ~375px, tablet ~768px, desktop ~1200px+).
- **Accessibility:** Use real semantic elements and sufficient color contrast for status/priority badges (e.g., don't rely on color alone to convey priority — pair color with text/icon) since this is called out in the Designer's brief as a first-class concern even though not CI-enforced.

## Validation Expectations

Concrete, mechanically-checkable (mirrors `.github/workflows/3-step.yml` and `scripts/validate-exercise.sh`):

1. `app/index.html`, `app/styles.css`, `app/project-data.json`, `.vscode/launch.json` all exist.
2. `app/index.html` contains (case-insensitive): `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`.
3. `app/styles.css` contains: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`.
4. `app/project-data.json` parses via `python3 -m json.tool` and contains (case-insensitive): `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`.
5. `.vscode/launch.json` parses via `python3 -m json.tool` and contains: `Run Project Pulse Dashboard`, `index.html`.

Manual/functional validation (not CI-enforced but required by the step's human review checklist):
6. Open **Run and Debug** → **Run Project Pulse Dashboard** → confirm the browser opens `http://localhost:5500/index.html` and shows the rendered Project Pulse dashboard with visible project cards (not a directory listing).
7. Confirm each card visibly shows status (as a badge), recent activity text, and a priority indicator.
8. Resize the browser to confirm the card grid reflows responsively.
9. Stop the preview server afterward (per step 3, item 6) before continuing/committing.

## Open Questions

1. **`serverReadyAction` mechanism with a Python command under a Node-type launch config** — is this reliably supported in the exact VS Code/Codespace version used here, or should Coder use `"type": "python"` with `module: "http.server"` instead? This needs empirical testing in the Codespace by whoever implements Phase 3; the plan flags the Node-wrapper approach as the literal-match to the step prompt's phrasing but notes a Python-native fallback.
2. **Exact status/priority vocabularies** — the brief doesn't mandate specific enum values (e.g., "On Track/At Risk/Blocked/Complete" vs. "Active/Paused/Done"); Coder/Designer should agree on a closed, small set before finalizing CSS modifier classes, otherwise Designer may style classes that don't match what Coder actually emits.
3. **Whether `index.html`'s inline `<script>` is acceptable style for the exercise**, versus a cleaner architecture with a separate `app.js` — given the strict file-scope validation described above, inline script is the safer choice, but this constrains code organization more than typical best practice would; worth flagging to the Orchestrator/learner as a deliberate exercise-driven tradeoff rather than a general recommendation for production code.
4. **Icon/visual assets** — the brief doesn't mention icons or images; Designer should confirm whether pure CSS (no external asset dependencies) is acceptable, which avoids network/asset-loading edge cases in a Codespace and is recommended for simplicity and determinism.