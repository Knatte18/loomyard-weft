# Support log — board-viewer

## Interview

### Turn 1 — exploration (autonomous)

The session ran autonomously with no advisors; every pick below is an auto-pick recorded in the question ledger.

- Board read: entry `board-viewer` (T21); no `Decision for board-viewer:` note on the board.
- Peers: `machine-registry-holds-catchup` changes `internal/boardengine` store and registry files, a text overlap with no design conflict;
  `board-prints-root-beam` (Layer B) changes how the prime prints and may touch the render, not the viewer's files.
- Two sonnet subagents mapped the board render pipeline and the `lym ide` VS Code writers.
- Findings that shaped the design:
  - `RenderToDisk` writes every render output and keeps a manifest for stale removal; the sync commits the Bolt tree; replay claims README and project files.
  - The README groups tasks by layer and notes by type, wraps each project in `<details open>`, and keeps bodies in design files.
  - `tasks.json` holds one task, `reed attach`, bound to `ctrl+alt+a` in the `// lym:begin` block; titles moved to console titles; folderOpen chains were dropped; nothing writes `launch.json`.
  - No markdown library in `go.mod`, no JavaScript runtime, no existing browser opener.

### Turn 2 — design picks

Scope, the data shape, the viewer asset, the opener and the README change were decided as in the ledger.

### Turn 3 — lens review

All six lenses returned a report.

- walkthrough 1 (BLOCKING, D6, two `beam start` tasks in one multi-root window start the wrong beam): fixed; `beam start` moves to the workspace files' `tasks` key with an explicit slug.
- walkthrough 2 (D1, projects have no section and no nesting in the data): fixed; D1 adds a project list with parent handles, D4 nests like the README and drops `project` from the kind filter.
- walkthrough 3 (D5, `--board-path` reaches another `board.html`): fixed; D5 states the flag in its bound.
- walkthrough 4 and bound-breaker 1 (D5, `xdg-open` can block in generic mode): fixed; D5 states the bound.
- bound-breaker 2 and existing-paths 3 (D6, more windows than the task pair miss a bound task): fixed; D6's bound names the unresolved prime, a git-tracked `tasks.json` and pre-upgrade windows.
- bound-breaker 3, existing-paths 1 and 2, interactions F1 (D2/D3, mixed lym versions, downgrade removal, empty replay of a render-only commit): fixed; D3 states the bound and D2 scopes its claim to one lym version.
- bound-breaker 4 (D5, `lymcwd.Resolve` refuses a subdirectory cwd): fixed; D5 resolves with `lymcwd.ResolveWithin`, and a criterion covers a subdirectory run.
- bound-breaker 5 (D7, GitHub app rendering of `<details>` unverified): rejected; the entry's operator asks for `<details>` for the GitHub app, and D7's rationale now cites that request.
- bound-breaker 6 and interactions F2 (D5, `lym board rerender` is internal and does not sync): rejected on the sync claim, since `Board.Rerender` runs the critical section that renders and spawns the sync; the internal audience is no denied step, so it stays the way forward, and D5 says it commits.
- board-scope 1 (decision, phone goal): already under Open risks; kept there for the parent.
- board-scope 2 (decision, prime `.vscode/tasks.json` tracked?): rejected; `git ls-files .vscode` on this branch is empty, so seeding reaches the prime, and D6's bound covers a repo that tracks the file.
- board-scope 3 (labels differ from the entry's wording): rejected; an auto-pick recorded in Q7.
- leanness 1, 2, 4, 6, 7 (repeats across Scope, Open risks, Goal, D4 and D7 rationale, Constraints): fixed; each stated once.
- leanness 3 (auto-mode assumptions repeat reasons): fixed; bullets cut to the bare pick with its id.
- leanness 5 (mill keys narrative in D6): fixed; kept as one Constraint line.
- leanness process note (only one fork spawned): rejected; six forks were spawned in one message and all returned.
- leanness D3 note (owner-set claim unverified): rejected; bound-breaker read `renderOwnerWrite` and confirmed it.

## Rejected alternatives

- `board-data.js` as raw `board.json` wrapped in `window.BOARD`: the viewer would recompute layers, handles and sections, duplicating the README's logic in JavaScript.
- Rendering bodies to HTML in Go: needs a new dependency or a hand-written renderer in Go; the JavaScript subset keeps the data plain.
- Writing the platform opener command straight into `tasks.json` with per-OS overrides: duplicates path resolution in VS Code config and gives no terminal verb.
- A Live Preview panel: an extension dependency the entry made optional.
- `beam start` for a task pair, running `lym beam start <parent beam>`: reads the parent per window and goes stale on a re-parent; not asked for.
- `beam start` in the prime's and each beam's folder `tasks.json`: a beam's multi-root window lists both folders, so the key would find two tasks and could start the root.
- Grouping the README by area: conflicts with the landed layer grouping; the viewer's area filter covers it.
- Copying `board.html` into the Bolt by a separate install verb: a fresh clone would lack it until someone ran the verb.
- `fabricengine.RegenerateHubWorkspace` writing the `beam start` tasks itself: extends fabric's IDE knowledge, which `PATTERN-fabric-boundary` forbids.
- The ide module writing the workspace files' `tasks` key after fabric's regeneration, preserved by fabric:
  a regeneration by any other caller, such as a fabric verb making a beam, would write a beam file with no `beam start` until the next seed.
- Collapsible Tasks, Notes and Done sections in every rendered board markdown file (the former D7): dropped once the parent judged today's README good enough on the phone.
- A `board open` task with `"reveal": "silent"`, relying on VS Code to reveal a failing task's terminal: that behaviour had no named source or release, so the task reveals always without focus.

## Review rounds

- Round 1: fixed F1–F6 and F9–F15; F7 and F8 went to the parent as `decision` findings.
  `beam start` moved behind a general workspace-content hook (D6), and `lym board open` resolves with `ResolveWithin` in the board pre-run (D5).
- Round 2: fixed F1–F7.
  D7 and the phone goal are gone after the parent's answer, the workspace splice moved into `internal/vscode` (D6), and `board open` reveals always without focus.

## Question ledger

- Q1 Data shape of `board-data.js` — auto-pick: a view projection from the README's model (D1). Keeps one source of placement.
- Q2 Where `board.html` comes from — auto-pick: embedded asset, written as a fixed render output (D2).
- Q3 Viewer grouping — auto-pick: by project and README section, not the demo's Tasks/Bugs/Enhancements, since the README regrouped after the demo.
- Q4 Body rendering — auto-pick: escaped Markdown subset in the page's JavaScript.
- Q5 How the task opens the browser — auto-pick: a `lym board open` verb with per-platform openers (D5).
- Q6 Where `beam start` lives — auto-pick after the lens review: the hub and beam workspace files, with an explicit slug, so a multi-root window holds one.
- Q7 Task labels and keys — auto-pick: `board open` and `beam start` after the `reed attach` style; `alt+shift+9` and `alt+shift+0` from the entry, clear of mill's `alt+shift+1` to `alt+shift+6`.
- Q8 Phone README — auto-pick: collapsible Tasks, Notes and Done per project (D7); raised under Open risks for the parent, since the entry's area and body grouping is outdated by landed work.
  Dropped in round 2 after the parent answered that today's README serves the phone.
- Q9 `Was ide-prime-setup` items already landed — auto-pick: out of scope; raised under Open risks.
