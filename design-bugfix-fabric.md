# Bugfix: fabric and config propagation

`bugfix-fabric` · Task · bug, fabric, config, reed

Bugfix only (operator, 2026-10-06): fix what each section below reports, with a regression test per fix that drives the real CLI wiring, not only fakes.
No new verbs, config keys or features; a reviewer treats any extension as a scope finding.
The board adds more bug sections here until the task starts.
One exception, by operator decision (2026-10-06): `fabric-add-adopt-remote` below is in scope whole, including `lyx fabric add` adopting existing origin branches.

## weft-lock-dir-drops-parent

### Bug

A strand launched in a pair whose weft worktree lacks the untracked `.weft/` lock directory gets no `LYX_PARENT`, silently.
`hubgeom.ResolveParent` reads the pair's origin record through `fabricengine.ReadOriginFor`, which requires both `_lyx/fabric/origin.json` and `.weft/`, and answers "no record" when either is missing.
`ResolveParent` then returns no parent, `hubgeom.reedGeometry` exports none, and nothing warns.

### Seen 2026-10-05

After moving `loom-bouncer-rounds` from another machine, its weft worktree was recreated by `lyx fabric reconcile` from the existing branch, which does not create `.weft/` (`lyx fabric add` does).
The driver, launched at 15:36:00, had no `LYX_PARENT`; `.weft/` appeared at 15:36:19 with the first weft write, and the Burler launched at 15:58 carries `LYX_PARENT='ly:orch'`.
The driver therefore could not reach its parent orchestrator.

### Fix

- Every path that creates or recreates a weft worktree creates `.weft/` (`reconcile`'s `weft_recreated` path at least).
- A present origin record with a missing lock directory is an error or a warning naming the gap, never a silent "no parent".

## config-reaches-parent

Seen 2026-10-04 twice: `pattern-init` first planned to keep PATTERN under `_lyx/` and needed a landing carry (dropped by moving PATTERN to warp), and `board-model`'s parent review found its D13 label list written into the pair's `_lyx/config/board.yaml`, which never reaches the parent.
Live config lives in weft, and a task's weft branch dies at landing, so any config change a task makes is lost unless the hub re-applies it by hand on main's weft.
Idea: either a landing step that carries `_lyx/config/` changes into the parent's weft (with a conflict stop), or task-made config changes declared in the PR description and applied by the hub as a checked step.

## fabric-add-adopt-remote

### Problem

Moving a running task to another machine needs a pair built from the task's existing warp and weft branches on origin.
Fabric handles an existing warp: `lyx fabric reconcile` adopts a warp worktree created outside lyx.
Finding the existing weft is the gap: `reconcile`, `checkout` and `add` probe only the local `refs/heads/<slug>-weft` (`weftBranchExists`).
When that ref is absent, `reconcile` forks a fresh dormant weft from the weft HEAD (`createDormantWeftForRawWarp`) and never looks at `origin/<slug>-weft`, so the run records and status are silently lost and the next push collides with origin.
Only `clone` adopts a weft branch from origin, and only for the prime.

### Workaround used on 2026-10-05

```
git -C <hub>/<repo>-weft branch --track <slug>-weft origin/<slug>-weft
git worktree add --track -b <slug> ../<slug> origin/<slug>
lyx fabric reconcile
```

With the local tracking weft branch in place, `reconcile` takes its `weft_recreated` path and wires junctions, portal and launchers.
`lyx batten run <slug>` from prime then resumed both the outer and the inner run, and the inner loom run continued from its producer row with a fresh agent.

The adopted pair's strands also lost their parent until the first weft write: see `weft-lock-dir-drops-parent` above.

### Proposal

When a branch exists on origin, fabric fetches it and checks it out as a tracking branch; it never creates a new local branch in its place.
`lyx fabric add <slug>` fetches `origin/<slug>` and `origin/<slug>-weft` when they exist, instead of refusing the warp branch as a diverged leftover.
Every other path that resolves a pair's weft branch (`reconcile`, `checkout`) checks `origin/<slug>-weft` the same way before forking a new one, as `clone` does.
