# Board README shows priority as its own sub-table per group

`board-priority-headings` · Note · enhancement, board

## Seen

The board README marks priority only with an italic `_(high priority)_` or `_(low priority)_` suffix after the title, which is easy to miss in a long table.

## Idea

Each README group (Ready, each layer, Bugs, Enhancements, Features) splits into one sub-table per priority, in the order high, normal, low, each under a fourth-level heading: `#### High priority`, `#### Normal priority`, `#### Low priority`.
A priority with no entries in the group gets no heading, and the title suffix goes away.

The operator chose plain headings (2026-10-10) over GitHub alert boxes, which VS Code's preview does not render, and over coloured emoji markers.
