# Fixer-report, round 1

All five findings fixed in commit `d8626e90`; `lym gate test ./internal/gitexec --tier 1` is green.
The five fixes touch the same two files, so they went in as one commit, not one per finding.

## Changes

- **F1** (fixed as proposed): `internal/gitexec/export_test.go` captures the production `startStallTimer` in `defaultStallTimer` and exposes it as `StartDefaultStallTimerForTest`.
  `TestRun_RemoteBounds` gains a row that starts it with a 1 ms bound and waits on a channel, with a 10 s guard, for the check;
  asserts stop returns false after the check ran;
  then starts it with a one-hour bound and asserts stop returns true.
- **F2** (fixed differently): the silent row's `ready` no longer reads the attempt number.
  Proposed: a per-attempt baseline of the run count taken at the first `ready` call, or a snapshot handed over at arming.
  Not made: a baseline taken at the first `ready` call misses a script run that lands before the first 5 ms tick, so the row would then wait out its deadline.
  Made: `ready` reports true once the script's run count exceeds the count at which the previous check fired (`firedAt`), so a retried git start that arms a check without running the script shifts nothing.
  The two-armed-checks assertion stays.
- **F3** (fixed as proposed): the answering row's `ready` sets a `checkRan` `atomic.Bool` when it returns true, and the row asserts the flag after `remoteAttempts`.
- **F4** (fixed as proposed): the unarmed row drops the `armed()` assertion and keeps the always-true `SetStallCheckForTest`, assigned to `_`.
- **F5** (fixed as proposed): the `TestRun_RemoteBounds` doc comment names the stall check seam among the process-global state.

## Gate repair

The gate found the comment atop the silent row's `firedAt` closure wrapped across two lines inside one sentence.
It is now two sentences, one per line, in a follow-up commit.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
