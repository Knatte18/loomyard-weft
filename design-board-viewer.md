# Interactive board view: a regenerated HTML file and a phone-friendly README

`board-viewer` · Note · enhancement, board, ui, ide

Operator 2026-10-04: the board needs an interactive view with filters, reachable without typing a lyx command each time.

- Rendering (operator, 2026-10-04: mechanical, never an LLM): `board.html` in `_board` is a fixed viewer, changed only when the viewer itself changes, with filters on kind, type and area labels, free-text search and expandable bodies.
  It loads a sibling `board-data.js` (`window.BOARD = {…}`), which lyx's Go code rewrites from `board.json` on every board write, committed in the same commit as README.md.
  A plain `fetch` of `board.json` does not work: browsers block it for a page opened from `file://`, while a `<script src>` loads.
  A working demo built on today's data was shown to the operator on 2026-10-04 (filters, grouping into Tasks/Bugs/Enhancements, expandable bodies, light and dark).
- Opening: `lyx ide` writes a VS Code task "Open board" into each worktree's VS Code config that opens `<hub>/_board/board.html` in the default browser, run from `Tasks: Run Task` or a keybinding; lyx resolves the path, so nothing is linked into the worktree and `PATTERN-hub-containment` holds.
  With Microsoft's Live Preview extension installed, the task may open it in a VS Code panel instead; VS Code alone shows HTML as source.
  A link from `_lyx` was considered and rejected: `_lyx` is the pair's weft, the board is hub-level, and file symlinks are not relied on (Windows).
- Phone: the board repo is private, so GitHub Pages needs a paid plan; instead make README.md easier to read in the GitHub app, grouped by type and area with collapsible `<details>` sections per area and per body.

Builds on `board-model`'s kind and labels.

