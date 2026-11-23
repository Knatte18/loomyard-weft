---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## Changes

- F1, F2, F5 (commit `1c17038b5`, `internal/landingshed/prreview.go`):
  - `wake` returns nil on a wake that read no status, so `haltOnEndedSession` runs only after `judge`.
    The `wake` and `wait` doc comments say so.
  - `prWaitState` gains `liveWarned`.
    A failed `SessionLive` read in `haltOnEndedSession` logs one Warn per episode and returns nil, and a successful read clears the flag.
  - The guard is `if !state.keepsSession`, without `|| rec.ReviewEnded`.
- F3, F4 (commit `ec0c399bf`, `internal/landingshed/prreview_wait_test.go`):
  - The dead-session row scripts `statusScript(open, open)` and advances the clock by `prPollInterval` when the session dies, so the halt comes from the branch after `judge`.
  - New row `AFailedLivenessReadKeepsWaitingAndLogsOnce` pins one Warn, `context.Canceled`, one start and no stop.
- Self-check follow-up (commit `6d62eb100`): the ended-at-entry row is renamed `AWaitEnteredOnAnEndedReviewKeepsWaitingThroughATransientPollFailure`.
  After F1 no status read reaches the `keepsSession` flag on that path, so the row cannot pin it and its old name claimed too much.
- `_lym/landing/summary.md` is updated for F1 and the self-check: the intro, the "Where the check runs" bullet, the "never watched" bullet and the Tests section.
  It is not committed, since `_lym` belongs to the loop.

The package gate (`lym gate test ./internal/landingshed`, tier0) passed after the F1 to F5 commits: 867 passed, 3 skipped, 0 failed.
The later rename commit was checked with `go vet` only.

## Self-check

- fix-consistency, LOW: `summary.md` Tests section says two rows. Fixed.
- fix-consistency, LOW: `summary.md` intro still says every open wake reads liveness. Fixed.
- fix-consistency, LOW: `summary.md` says a record whose review has ended is never watched. Fixed, now names `keepsSession` false.
- fix-proposal, LOW: `summary.md` intro and Tests section stale. Duplicate of the three above, fixed.
- fix-proposal, LOW: the review's claim that `endReview` is the only writer of `ReviewEnded` is false, since `finalize_pr.go:565` also writes it.
  The change is still sound, because `judge` ends the call on `ReviewEnded` before the check runs, so the claim is not repeated.
  Not a code change.
- fix-code-correctness, LOW: the ended-at-entry row cannot pin `keepsSession` after F1. Fixed differently: the row is renamed to what it observes.
  Dropping the `keepsSession` argument at the entry sites is a larger design change and was not made.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
