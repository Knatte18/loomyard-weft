# Round 2 fixer report — plan

Every finding is fixed. `lym loom validate-plan` answers `ok: true` with informational findings only.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Card 10 re-signs `saveRunState` as `func saveRunState(geom Geometry, st *State, leaseHeld bool) error`.
  With `leaseHeld` set it skips the acquire; with it unset it acquires `runplan.AcquireStateMutation` first, which card 10 now lists under Uses.
  Card 12 says the begin saves with `leaseHeld` true before it releases the lease, and every other save passes false.
  `TestRun` gains a begin row over a real scratch directory; after the self-check it runs under a test deadline and probes the lease with `lock.AcquireWriteLockWithin`.
- **F2 (MEDIUM), fixed as proposed.**
  Card 10's `endAttempt` pass clears `StopRecord`.
  Card 13 replaces its clearing sentence with the rule for all three sites: `endAttempt`, `recordDoneWithoutSpawn` and a passing verify re-run.
  `TestRun_Reentry` gains the row for the last batch whose final record commit fails.
  Card 5's `Stop` field text names the same clearing sites, so it agrees.
- **F3 (MEDIUM), fixed differently.**
  Card 13 creates `checkStoppedBatch` and `recordDoneWithoutSpawn` and names `reentry`'s fields: the batch, the start rung, and whether the verify is next.
  I did not make two parts of the proposal:
  - Not done: run the head check on re-entry and share it with `endAttempt`.
    A repair the parent or arbiter commits after the stop moves HEAD past the last report's `head_sha`, so a head check would refuse the no-spawn path D7 exists for.
    So the shared helper is card 10's `checkCleanTree(git websterengine.Git, worktree string) ([]string, error)`, and the head check stays in `endAttempt` alone.
  - Not done: return only failing reasons from `checkStoppedBatch`.
    `runplan.PostBatchChecks` returns deviations (`internal/runplan/postbatch.go:241`), and a done record from `endAttempt` keeps them.
    So `checkStoppedBatch` returns `(deviations, reasons []string, err error)`, and `recordDoneWithoutSpawn` takes the deviations and records them with the departures and card trail.
    Card 13's Uses gains `runplan.BatchDepartures` for this.
- **F4 (MEDIUM), fixed differently.**
  - Card 13 creates `resumeResult` and re-signs `resumeStrands` to return `(resumeResult, error)`.
  - Its step 4 states the continuation rules for a recordable, failed, lost and needs-parent attempt.
  - `TestRun_Resume` gains the rung 0 to rung 1 row and the lost same-rung retry row.
  - The begin-reuse rule and the refusal naming `reentryWayForward` go in card 13, not card 12.
    Card 13 creates `reentryWayForward` and edits `Run`, so card 12 cannot name it.
  - After the self-check, card 12's `runBatch` bullet says the same-rung retry allowance is read from the batch's recorded attempts.
- **F5 (LOW), fixed differently in one point.**
  Card 12 states the `outcome.yaml` keys: `outcome: done`, an empty `stuck_reason` and `batches_done`.
  Card 23's `RunInFlight` decodes exactly those keys and refuses an unknown one.
  Differently from the proposal, it reads `stuck` as in flight rather than as an error.
  That keeps Webster's two legal values (`internal/websterengine/outcome.go:54`) and card 23's existing `stuck` row.
- **F6 (LOW), fixed as proposed.**
  Card 18's wire bullet sets `WeaverRun` to `weaverengine.Run` explicitly.
  Card 16 drops the default, so the adapter calls only the injected `run`.
- **F7 (LOW), fixed as proposed.**
  The duplicate attempt-number row is gone from `TestRun_Resume`.

## Self-check

- fix-claims, MEDIUM: the `checkTreeAndHead` signature can't drive `DirtyPaths` and `MergeRejection`, which need a worktree and a `ParentBranchFunc`.
  Fixed with my own change: the helper became `checkCleanTree(git, worktree)`, with no parent-branch parameter, as F3 above describes.
- fix-consistency, MEDIUM: card 13 changes `runBatch`'s retry allowance, but card 12's `runBatch` bullet still reads it as a counter within one call.
  Fixed as proposed, in card 12's bullet.
- fix-proposal, MEDIUM: the head check on re-entry refuses a repair committed after the stop.
  Fixed with my own change: re-entry drops the head check, as F3 above describes.
- fix-proposal, MEDIUM: the no-spawn done record loses its deviations and departures.
  Fixed as proposed: `checkStoppedBatch` returns the deviations, and `recordDoneWithoutSpawn` records them.
- fix-proposal, LOW: the lease row pins a regression only as a hang.
  Fixed as proposed: the row runs under a test deadline and probes with `lock.AcquireWriteLockWithin`.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached me this round.
