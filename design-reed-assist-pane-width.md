# Reed gives the Codex assist reviewer pane a fixed width instead of half beside Claude

`reed-assist-pane-width` · Note · bug, reed, codex

## Seen (operator, 2026-11-08)

When a Code-Review round runs a Codex assistant reviewer beside the Claude reviewer, reed puts the Codex pane in a column of its own beside the Claude reviewer's pane, and that column keeps a fixed width: it does not follow when the window or the Claude pane is resized.

## Idea

The two reviewer panes share their space evenly and scale together: each takes half the width, on every layout render and every resize, as other side-by-side strands do.
