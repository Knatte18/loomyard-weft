# Fixer report — round 1

## Changes

- **F1** (commits `2c5550b6`, renamed row in the follow-up commit): new `internal/loomcli/watchfilechanges_test.go`, tier 0, parallel.
  Rows: a write to the watched file delivers a change; a write to a sibling delivers nothing; a missing directory returns a nil channel and a callable stop; `stop` ends the watch and may be called twice; the channel holds one buffered value.
  `TestPullRequestDeps_RecordChangesWatchesTheRunsPullRequestRecord` writes `shedrun.PullRequestRecord(location, shedrun.SelfRunID)` and reads the `RecordChanges` channel.
  Fixed with an own change on one point: the review asked that two writes before a read merge into one value.
  A write to an existing file can emit several events, so a behavioural merge assertion is racy.
  The row asserts `cap(changes) == 1` and is named for that (`TheChannelHoldsOneBufferedValue`), not for merging.
- **F2** (commit `567ead27`): `changeDuringFirstSleep(elapsed)` replaces the inline setup of the standing-clock row, and two rows join it: 400ms elapsed expects sleeps `[1s, 600ms]`, 1s elapsed expects `[1s]`.
  The generic sleep-bounds loop failed on the 600ms rest, so a `restsFloorRemainder` row field exempts that row from the one-second minimum, and the loop now also rejects a sleep of zero or less.

## Self-check

- fix-code-correctness, LOW, merge row asserts only the buffer size: fixed with my own change (renamed, as above).
- fix-code-correctness, LOW, absence checks use a 200ms window: disputed, every fix worse (see Disputed).
- fix-consistency, LOW, parallel missing-directory row emits a global Warn: disputed, premise false (see Disputed).
- fix-proposal, MEDIUM, merge row passes with the defect in place: fixed with my own change (renamed, as above); a deterministic merge test needs a fake events source in production code.
- fix-proposal, LOW, absence checks timing-based: same finding as the correctness lens, disputed.
- fix-proposal, LOW, `restsFloorRemainder` loosens the bound for the whole row: disputed, premise false (see Disputed).
- fix-affected-set, LOW, Beam-Await watch unpinned: deferred (see Deferred).
- fix-affected-set, LOW, merge row asserts only capacity: same finding as above, fixed with my own change.

## Disputed

- **Absence checks (correctness, proposal lenses)**, ground `every fix worse`: the premise holds, but the false pass is the only error direction, and a deterministic absence check needs a fake events source injected into `watchFileChanges`, an abstraction the production code does not otherwise need.
- **Parallel Warn (consistency lens)**, ground `premise false`: the only other `logcapture` user in the package outside serial tests is `TestIdleReleaseLogLevels` (`looplock_test.go:102`), which has no `t.Parallel()` at its top level, so it runs before the parallel tests resume and cannot see the Warn.
- **`restsFloorRemainder` loosening (proposal lens)**, ground `premise false`: the flag is set on the 400ms row alone, and that row's `check` pins its sleeps exactly to `[1s, 600ms]`, so a stray sub-second interval sleep fails there.

## Deferred

- **Beam-Await watch (affected-set lens)**: `openLandRequestWatch` in `internal/loomcli/pawlwiring.go` is the second caller of `watchFileChanges`.
  The review's `fix` key scopes the test to the helper and the `pullRequestDeps` seam, and the helper's behaviour is now pinned for both callers.
  A test of `openLandRequestWatch` is a separate coverage item for the orch to board if wanted.

## Gate

`lym gate test ./internal/landingshed ./internal/loomcli --tier 0`: all pass (1823) after F2; re-run after the rename below.
