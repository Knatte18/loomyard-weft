```yaml
lens: board-scope
outcome: reported
```

Scope: checking how the board-viewer record fits the board (goal points, peers, decision notes).

**Findings**

- **MEDIUM**, class `decision`, Open risks / D7. The entry's phone point asks for README sections "grouped by type and area with collapsible `<details>` sections per area and per body". D7 builds folding of Tasks, Notes and Done only, and passes area grouping to the viewer. The record raises this under Open risks, so it is not silent. It still drops a stated goal point. Question for the parent: does folding sections over the landed layer grouping, with bodies left in `design-<slug>.md`, meet the phone goal, or is a per-area grouping inside each project still wanted?
- **LOW**, class `decision`, Scope Out / Open risks. The `Was ide-prime-setup` part also says lym should seed the prime's `.vscode/` and assume none exists beforehand. The record covers titles, folderOpen and `launch.json`, but not this seed-from-scratch point. Today `writeVSCodeConfig` skips when `tasks.json` is git-tracked. The prime's hand-copied, millhouse-derived `.vscode/` may therefore be tracked, and then D6's tasks would never reach the prime. Question for the parent: is the prime's `.vscode/tasks.json` still tracked in the hub's warp? If it is, should this task stop tracking it or overwrite it?
- **LOW**, D6. The entry names the task labels "lym: beam start" and "lym: open board". The record picks `beam start` and `board open`. It lists this as an auto-pick, a choice the writer may make, so this is not a contradiction. It is noted only because the entry's wording differs.

**Goal points**

- Fixed `board.html` that loads `board-data.js`, rewritten on every write and committed with the README: met (D1–D3).
- Filters on kind, type and area, free-text search, expandable bodies: met (D4).
- Grouping into Tasks/Bugs/Enhancements and light/dark, as in the demo: light/dark is met. The grouping is changed to project and section, as an auto-pick.
- No `fetch`; data comes in through `<script src>`: met (D2).
- A VS Code task "Open board" that opens the viewer in the default browser, with lym resolving the path and nothing linked: met (D5, D6).
- Live Preview ("may"): left out, an optional item, so not a conflict.
- Phone collapsible README: partly met; see the first finding.
- Running section as one list with no priority groups (2026-10-12): untouched and consistent with the record.
- `Was ide-prime-setup`: the beam start task and the Alt+Shift+0/9 keys through the same block writer are met (D6). Titles, folderOpen and `launch.json` are outdated by landed work and raised under Open risks. Seeding with no prior `.vscode/` is covered by the second finding.
- The reed-view D15 note that keys go through the same writer: met (D6).

**Entries, peers and commits checked**

- `board-prints-root-beam` (Layer B): no conflict. Its record changes `parent_beam` printing and the `--text` P0 row, not the render outputs or the viewer.
- `machine-registry-holds-catchup` (running, Webster): touches `boardengine` store and registry. This is text overlap only, no conflict.
- `codex-standing-instructions`, `sensitive-path-commit-guard`, `gate-builds-testmap`, `edit-loop-detector`, `stencil-pin-per-run`, `review-rounds-converge`, `per-machine-branch-prefix`: no overlap with the board render or `ide` tasks, so none.
- Parent commits: `lym loom peers` lists none.

**Decision notes:** the board list shows no `Decision for board-viewer:` note, and the record has no `Added after Discussion` entries.

**Tool calls**

- Grepped `_bolt/decision-board-prints-root-beam.md` for render and README terms. My context had that peer's slug but not its record, and I needed it to check for a render or viewer conflict.
