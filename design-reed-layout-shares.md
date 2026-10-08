# reed: strands state their size; parallel agents share the window evenly

`reed-layout-shares` · Note · enhancement, reed

## Why

reed has one layout rule (`internal/reedengine/doc.go`, "The layout rule"): the strands stack in insertion order, every strand but the bottom-most takes `collapsed_rows` rows, and the bottom-most takes every remaining row.
It fits a run whose newest agent is the one that matters, and nothing else:

- `runs-overview` wants a small pane under the orch, sized to its content, with the orch above keeping the space; today the orch would collapse to `collapsed_rows` and the overview would grow.
- `multi-agent-producer` runs N agents in parallel, none more important than the others; each needs a large share of the window, and none may collapse.

## Proposal (operator, 2026-10-08)

- A strand states how it is sized instead of taking its size from its position.
- At least two kinds: a fixed or content-sized strand (the overview: as few rows as show all its lines), and a sharing strand that splits the remaining space evenly with the other sharing strands.
- The agents of one multi-agent step share the window evenly, side by side or stacked; which of the two is open.
- The collapsed stack of finished agents may stay as one kind among these, for strands kept only for reference.

## Open

- Side by side (columns) or stacked rows for parallel agents, or a rule by window width.
- Who sets a strand's kind: the spawning producer through its seat table (`multi-agent-producer`), or reed by role.
- How `runs-overview`'s content-sized pane learns its height as runs come and go, and whether reed re-applies the layout on each change.
