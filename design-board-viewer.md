# Interactive board view: a regenerated HTML file and a phone-friendly README

`board-viewer` · Note · feature, board, ide, orch

Operator 2026-10-04: the board needs an interactive view with filters, reachable without typing a lym command each time.

- Rendering (operator, 2026-10-04: mechanical, never an LLM): `board.html` in `_board` is a fixed viewer, changed only when the viewer itself changes, with filters on kind, type and area labels, free-text search and expandable bodies.
  It loads a sibling `board-data.js` (`window.BOARD = {…}`), which lym's Go code rewrites from `board.json` on every board write, committed in the same commit as README.md.
  A plain `fetch` of `board.json` does not work: browsers block it for a page opened from `file://`, while a `<script src>` loads.
  A working demo built on today's data was shown to the operator on 2026-10-04 (filters, grouping into Tasks/Bugs/Enhancements, expandable bodies, light and dark).
- Opening: `lym ide` writes a VS Code task "Open board" into each worktree's VS Code config that opens `<hub>/_board/board.html` in the default browser, run from `Tasks: Run Task` or a keybinding; lym resolves the path, so nothing is linked into the worktree and `PATTERN-cwd-resolution` holds (it absorbed `PATTERN-hub-containment` in `04aafa88c`).
  With Microsoft's Live Preview extension installed, the task may open it in a VS Code panel instead; VS Code alone shows HTML as source.
  A link from `_lym` was considered and rejected: `_lym` is the pair's weft, the board is hub-level, and file symlinks are not relied on (Windows).
- Phone: the board repo is private, so GitHub Pages needs a paid plan; instead make README.md easier to read in the GitHub app, grouped by type and area with collapsible `<details>` sections per area and per body.
  Built by `board-readme-per-project` (2026-10-28): `README.md` is the root project's file and every project renders to `project-<slug>.md` beside it, research notes first; the collapsible `<details>` sections per area and per body are not built.

Builds on `board-model`'s kind and labels.

Operator 2026-10-12: the README's Running section is one list, with no priority groups.


## Was `ide-prime-setup`

Operator 2026-10-04: the prime's `.vscode/` was copied by hand from millhouse (title "LYMHUB", millhouse keys, a mill `launch.json`); lym should seed it instead, and the setup should fit the orchestrator workflow.
Assume no `.vscode/` exists beforehand, or that it may be deleted and re-seeded by lym.

- Window title: `<SHORTNAME>: <name>` from `.lym-shortname`, upper-cased: `LY: prime` for the prime, `LY: <slug>` for pairs (today `internal/vscode/config.go` writes the bare slug).
- Prime folderOpen chain: `reed up` then `reed attach` only; no `reed add claude` (it adds a plain Claude strand the orchestrator workflow does not use).
- Prime tasks run on demand: "lym: beam start" (`lym beam start`, which replaced the removed `lym orch start`: launches or attaches the orchestrator and takes the terminal) and "lym: open board" (opens the board viewer, see `board-viewer`).
- Keybindings: lym writes a `// lym:begin` … `// lym:end` block into the user `keybindings.json`, the way mill writes its `// mill:begin` block (mill holds Alt+Shift+1–6), e.g. Alt+Shift+0 for orch start and Alt+Shift+9 for open board, binding `workbench.action.tasks.runTask` to the task labels.
- No `launch.json`: mill used it to run Python scripts through the debugger; lym is a binary, so tasks are the mechanism.
- Pairs keep their current chains (driven: attach only).


From `reed-view` Discussion (D15, 2026-10-08): `lym ide spawn` rewrites the whole `// lym:begin` … `// lym:end` block in the user `keybindings.json` on every run, for the reed Alt keys.
This note's keys must be added through the same writer, or each spawn wipes them.
