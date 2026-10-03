# Card 11 — batten-teardown-through-composite

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli#teardownRefusal`
- `internal/battenshed#TeardownDeps`
- `internal/battencli/wire_test.go`

**Uses:**
- `plan:internal/pairteardown#Teardown`
- `plan:internal/pairteardown#New`
- `plan:internal/pairteardown#Request`
- `plan:internal/pairteardown#Teardown.EndSession`
- `plan:internal/pairteardown#Teardown.RemovePair`
- `plan:internal/fabricengine#ErrPairNotFound`
- `internal/battenshed#NewWorktreeTeardown`

**Intent:**
Batten's Worktree-Teardown row tears the pair down through the composite.
It keeps its one producer and its two-closure `TeardownDeps`, which already sequence session shutdown strictly before removal (Batten Bookend Invariant).

In `wire`, both closures build `pairteardown.New(location)` with `Request{Slug: slug, Force: false, Remote: true, QuietWait: 0, RefuseWhenBusy: false}`.
- `Shutdown` calls `EndSession` and returns its `AbandonedSession`, recorded on the receiver as today.
  The task-worktree-gone skip goes: `EndSession` ends the session by name when the worktree is gone, instead of leaving it to the watchdog.
  A driver still live after the InnerRun done arm's `driver_exit_grace_s` is ended here, and the abandoned-session record still comes from `Down` exactly as today.
- `Remove` calls `RemovePair`.
  `errors.Is(err, fabricengine.ErrPairNotFound)` is the done post-condition: log it and return nil.
  The `PairSiblingRemnant` refusal and the `RemovePairBranch` re-entry path go, since `Remove` now finishes those states.
  A non-empty `RemoteBranchError` still halts resumable as today.
- `teardownRefusal`'s `ErrPairSiblingDirty` arm is reworded for the narrowed meaning: run records are committed by the teardown itself, so a refusal now means content outside the record paths in the pair's sibling.
  The remedy names inspecting and committing or removing that content by hand, then resuming, and never `--force`.

`TeardownDeps`' doc comment says both closures are expected to call the pair-teardown composite's two phases.

Bound: batten's teardown gains no new deletion and no new skip.
It still never forces, and the only driver it ends is one still live after its configured grace, as today.

Tests in `wire_test.go`:
- `TestWire_TeardownRefusesAHalfTornPair` becomes a test that a half-removed pair's teardown no longer refuses by name; it reaches the composite's removal and does not use `PairSiblingRemnant`.
- `TestWire_TeardownIsIdempotentAgainstAnAlreadyRemovedWorktree` asserts a not-found removal reports done.
- `TestTeardownRefusal_RecordsRemedyNamesCommitRecordsNeverForce` is updated to the new remedy text, still asserting `--force` is never named.

**ImpactSummary:** Batten's two teardown closures change bodies; the producer, its stuck reasons and its prime-lock handling are unchanged.

**Commit:** `11: batten teardown goes through pairteardown`
