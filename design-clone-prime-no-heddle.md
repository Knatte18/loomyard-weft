# fabric clone leaves the prime without rendered heddle

`clone-prime-no-heddle` · Note · bug, fabric, heddle

Imported from [issue #230](https://github.com/Knatte18/loomyard/issues/230).

## Seen

In the DrillOpPlan hub, set up on 2026-11-10, the prime worktree had no `CLAUDE.local.md` after `lym fabric clone`, though heddle held content: `lym heddle show claude-file` printed lym's shipped rules (batching calls, the order of a round of edits).
Every worktree lym made later, beams and task worktrees, got the file rendered.
The Models hub showed the same: its prime anchor had no `CLAUDE.local.md`, while its beam and task worktrees, such as `support-for-dop`, had one.
So the orch and any other session in the prime ran without heddle's rules.

## Expected

`lym fabric clone` renders heddle into the prime it creates, as lym does for every worktree it makes.

## Workaround

The operator ran `lym heddle render` in the prime by hand.
