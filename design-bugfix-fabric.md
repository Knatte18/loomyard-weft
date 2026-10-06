# Bugfix: fabric and config propagation

`bugfix-fabric` · Task · bug, fabric, config, reed

Bugfix only (operator, 2026-10-06): fix what each section below reports, with a regression test per fix that drives the real CLI wiring, not only fakes.
No new verbs, config keys or features; a reviewer treats any extension as a scope finding.
The board adds more bug sections here until the task starts.

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
