# Card 11 — The `Preflight` row reconciles a stale hub before its clean check

**Create:**
- `internal/preflightshed#newPreflightWith`
- `internal/preflightshed#reconcileRefusal`

**Edit:**
- `internal/preflightshed#preflightProducer`
- `internal/preflightshed#NewPreflight`
- `internal/preflightshed#preflightProducer.Call`
- `internal/preflightshed/preflight_integration_test.go`
- `internal/preflightshed/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/hubgeom#ReconcileGeometry`
- `internal/hubreconcile#Ensure`
- `internal/hubreconcile#WorktreeError`
- `internal/hubreconcile#LockTimeoutError`
- `internal/preflight#HubPresent`
- `internal/preflight#Check`

**Intent:**
Decision 10: the loom recipe's `Preflight` row checks the same stamp and calls the same entry point, as a safety net for a run whose start verb predates the binary change or that reaches `Preflight` without one (a `goto` re-entry, a driver started before the binary changed).

`preflightProducer` gains an `opts hubreconcile.Options` field.
`newPreflightWith` builds it with told options.
`NewPreflight` calls it with the zero `Options`, so its signature and `shedrecipe`'s `preflightEntry` are unchanged.

`Call` runs the reconcile after `entryErr` and before `preflight.Check`, so the committed reconcile leaves the tree clean for the worktree-clean check:
1. When `preflight.HubPresent(p.cwd)` and `hubgeom.ReconcileGeometry` report a hub, it calls `hubreconcile.Ensure(geom, p.opts)`.
2. A nil error proceeds to `preflight.Check` as today.
3. Any error is logged with `logger.Warn` and returns `shedengine.Stuck` with `OutputPointer{Reason: reconcileRefusal(err)}`; a cancelled context still returns `cancelErr` first.

`reconcileRefusal` maps the error to a reason:
- A `*hubreconcile.WorktreeError` names the worktree, file and cause, with `; way forward: fix <file> (or run "lyx config <module>" on it), then run "lyx loom resume" in the task worktree`.
  The `lyx config` clause is dropped when the error names no module.
- A `*hubreconcile.LockTimeoutError` says another command holds the hub's reconcile lock, with `; way forward: run "lyx loom resume" in the task worktree once the other reconcile ends`.
- Any other error gives its message with the first way forward's tail.

A failure in an unrelated pair stops the row too (Decision 9's bound): no run in the hub can start or resume until the named file is fixed, while every other command still runs.
Loom-Preflight's retired-key refusal and its `lyx config reconcile --apply` way forward stay unchanged.

`contracts/specs/refusal-spec.md`'s shed section gains two rows in the same commit (`PATTERN-refusal-way-forward`):
- "Preflight: hub config reconcile failed": the row finds a stale stamp and the walk fails on a config file in any worktree, or on a commit. Class: correctness halt. Way forward quotes the first reason's tail.
- "Preflight: hub reconcile lock busy": the hub lock is still held when the wait runs out. Class: transient. Way forward quotes the second.

`doc.go` gains a paragraph on the stamp check, its place before `preflight.Check`, and its two Stuck reasons.

Tests in `preflight_integration_test.go` (its fixture hub plus a pair), built through `newPreflightWith` with a 50 ms `LockWait`:
- An absent stamp and a committed retired `master_base` key in the task worktree's `batcher.yaml` give `Done`, with the key removed, the stamp written and the tree clean.
- An absent stamp and an unparseable `loom.yaml` in another pair give `Stuck`, with a reason naming that file, `lyx config loom` and `lyx loom resume`, and no stamp.
- A held `LockPath` gives `Stuck`, with a reason naming `lyx loom resume`, and no stamp.

**ImpactSummary:** The `Preflight` producer gains one gated reconcile ahead of its unchanged check; a hub with a fresh stamp pays one file read.

**Commit:** 11: preflight row: reconcile a stale hub's config before the clean check
