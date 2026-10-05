# Config changes a task makes never reach the parent

`config-reaches-parent` · Note · bug, config, fabric, loom

Seen 2026-10-04 twice: `pattern-init` first planned to keep PATTERN under `_lyx/` and needed a landing carry (dropped by moving PATTERN to warp), and `board-model`'s parent review found its D13 label list written into the pair's `_lyx/config/board.yaml`, which never reaches the parent.
Live config lives in weft, and a task's weft branch dies at landing, so any config change a task makes is lost unless the hub re-applies it by hand on main's weft.
Idea: either a landing step that carries `_lyx/config/` changes into the parent's weft (with a conflict stop), or task-made config changes declared in the PR description and applied by the hub as a checked step.
