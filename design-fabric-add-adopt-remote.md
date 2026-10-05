# fabric fetches an existing remote warp and weft branch instead of creating new ones

`fabric-add-adopt-remote` · Note · enhancement, fabric

## Problem

Moving a running task to another machine needs a pair built from the task's existing warp and weft branches on origin.
Fabric handles an existing warp: `lyx fabric reconcile` adopts a warp worktree created outside lyx.
Finding the existing weft is the gap: `reconcile`, `checkout` and `add` probe only the local `refs/heads/<slug>-weft` (`weftBranchExists`).
When that ref is absent, `reconcile` forks a fresh dormant weft from the weft HEAD (`createDormantWeftForRawWarp`) and never looks at `origin/<slug>-weft`, so the run records and status are silently lost and the next push collides with origin.
Only `clone` adopts a weft branch from origin, and only for the prime.

## Workaround used on 2026-10-05

```
git -C <hub>/<repo>-weft branch --track <slug>-weft origin/<slug>-weft
git worktree add --track -b <slug> ../<slug> origin/<slug>
lyx fabric reconcile
```

With the local tracking weft branch in place, `reconcile` takes its `weft_recreated` path and wires junctions, portal and launchers.
`lyx batten run <slug>` from prime then resumed both the outer and the inner run, and the inner loom run continued from its producer row with a fresh agent.

## Proposal

When a branch exists on origin, fabric fetches it and checks it out as a tracking branch; it never creates a new local branch in its place.
`lyx fabric add <slug>` fetches `origin/<slug>` and `origin/<slug>-weft` when they exist, instead of refusing the warp branch as a diverged leftover.
Every other path that resolves a pair's weft branch (`reconcile`, `checkout`) checks `origin/<slug>-weft` the same way before forking a new one, as `clone` does.
