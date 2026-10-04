# Interactive board view: a regenerated HTML file and a phone-friendly README

`board-viewer` · Notes · feature

Operator 2026-10-04: the board needs an interactive view with filters, reachable without typing a lyx command each time.
Idea: every board write regenerates, next to README.md, one self-contained `board.html` (data embedded, no server) at a fixed hub path (`<hub>/_board/board.html`), with filters on kind, type and area labels, free-text search and expandable bodies; the operator opens it from a browser bookmark.
Optionally the hub VS Code workspace lists `_board` as a folder (a workspace entry, not a junction into a worktree, so `PATTERN-hub-containment` holds).
Phone: the board repo is private, so GitHub Pages needs a paid plan; instead make README.md easier to read in the GitHub app, grouped by type and area with collapsible `<details>` sections per area and per body.
Builds on `board-model`'s kind and labels.
