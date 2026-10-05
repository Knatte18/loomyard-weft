# A weft worktree without .weft/ silently drops LYX_PARENT from every strand

`weft-lock-dir-drops-parent` · Note · bug, fabric, reed

## Bug

A strand launched in a pair whose weft worktree lacks the untracked `.weft/` lock directory gets no `LYX_PARENT`, silently.
`hubgeom.ResolveParent` reads the pair's origin record through `fabricengine.ReadOriginFor`, which requires both `_lyx/fabric/origin.json` and `.weft/`, and answers "no record" when either is missing.
`ResolveParent` then returns no parent, `hubgeom.reedGeometry` exports none, and nothing warns.

## Seen 2026-10-05

After moving `loom-bouncer-rounds` from another machine, its weft worktree was recreated by `lyx fabric reconcile` from the existing branch, which does not create `.weft/` (`lyx fabric add` does).
The driver, launched at 15:36:00, had no `LYX_PARENT`; `.weft/` appeared at 15:36:19 with the first weft write, and the Burler launched at 15:58 carries `LYX_PARENT='ly:orch'`.
The driver therefore could not reach its parent orchestrator.

## Fix

- Every path that creates or recreates a weft worktree creates `.weft/` (`reconcile`'s `weft_recreated` path at least).
- A present origin record with a missing lock directory is an error or a warning naming the gap, never a silent "no parent".
