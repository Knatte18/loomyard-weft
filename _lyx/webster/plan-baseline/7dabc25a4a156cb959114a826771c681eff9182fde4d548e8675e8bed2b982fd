# Card 10 — `lyx batten run` reconciles before and after creating its pair

**Edit:**
- `internal/battencli#battenCLI.arm`
- `internal/battencli#battenCLI.wire`
- `internal/battencli/lifecycle_integration_test.go`

**Uses:**
- `internal/hubgeom#ReconcileGeometry`
- `internal/hubreconcile#Ensure`
- `internal/hubreconcile#Options`
- `internal/fabricengine#WorktreePath`

**Intent:**
Decisions 3 and 4 for `lyx batten run`.

First call: `battenCLI.arm`, for `verb == "run"` only, calls `hubreconcile.Ensure(geom, hubreconcile.Options{})` right after `lyxcwd.Resolve` and before `armAt`, so it runs before the run is seeded, before `wire` loads any module config and before the `Worktree-Create` row can fork the pair.
The fork then reads the prime's reconciled, committed records branch.
Its error is returned, so the pre-run's existing `output.Err` path prints it.
`ArmAt` (the `lyx shed` path), `step`, `status`, `pause` and `goto` never call it.

Second call: in `wire`, the `CreateWorktree` closure, after `Topology.Add` succeeds, and also on the already-complete skip (the adopted pair), calls `hubreconcile.Ensure` with `Options{Pair: fabricengine.WorktreePath(location, slug)}`.
It returns the call's error, so a failure stops the row and the run with the message naming the file and the way forward, and a lock timeout the same way (Decision 4).
That pair-scoped call reconciles and commits the new pair under the hub lock whatever the stamp says, which covers a pair that forked a prime the walk skipped as mid-merge, or one another holder's walk had not yet committed.
It costs one pair's config read, and no commit when nothing changed.
Bound: it touches only that pair's `_lyx/config` files, and an `Add` that fails is returned as today without the call.

Tests in `lifecycle_integration_test.go`, one new test on a hubforge hub:
- Remove the stamp and write a fresh stamp carrying the running key, so the first call is a no-op.
- Commit a retired `master_base` key into the prime's records-side `batcher.yaml`, which the fork inherits.
- After the `Worktree-Create` row has run (driven the way the file's existing create rows drive it), the new pair's `batcher.yaml` lacks the key and its records worktree is clean, which only the unconditional pair call can have done.
- A second row runs `arm` for `run` from one of the hub's pairs against an absent stamp.
  `armAt`'s non-prime refusal (`RefuseUnlessPrime`) stops it before the auto-seed, so the returned refusal, a written stamp and no seed for the run id in either worktree pin that the reconcile runs ahead of `armAt` and its seed.

**ImpactSummary:** `lyx batten run`'s pre-run gains one gated call and its `Worktree-Create` closure one pair-scoped call; every other batten verb is unchanged.

**Commit:** 10: batten: reconcile the hub before creating a pair, and the new pair after
