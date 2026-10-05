# fabric add adopts an existing remote warp and weft branch pair

`fabric-add-adopt-remote` · Note · enhancement, fabric

## Problem

Moving a running task to another machine needs a pair built from the task's existing warp and weft branches on origin.
`lyx fabric add <slug>` cannot do that.
It adopts only an existing local weft branch; the warp side always runs `git worktree add -b <branch>` from the acting worktree's HEAD.
A local warp branch is refused with `ErrBranchExists`, and an origin warp branch that is not an ancestor of HEAD is refused by `probeWarpLeftover` as a diverged leftover.

## Workaround used on 2026-10-05

```
git -C <hub>/<repo>-weft branch --track <slug>-weft origin/<slug>-weft
git worktree add --track -b <slug> ../<slug> origin/<slug>
lyx fabric reconcile
```

`reconcile` then recreates the weft worktree from the existing branch (`weft_recreated`) and wires junctions, portal and launchers.
`lyx batten run <slug>` from prime resumed both the outer and the inner run, and the inner loom run continued from its producer row with a fresh agent.

## Proposal

An adopt path in `lyx fabric add` (or a dedicated verb) that tracks both origin branches when they exist, so moving a run between machines is one command.
