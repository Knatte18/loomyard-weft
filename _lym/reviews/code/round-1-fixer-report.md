# Fixer-report, code review round 1

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  In `internal/gatecli/test_integration_test.go` the orphan subtest is now a table over the shell's exit code, rows `FailingExit` (134, failed) and `ZeroExit` (0, passed), so the `exec.ErrWaitDelay` to nil mapping in `runTiedChild` is pinned.
  Commit `64636804`.
- **F2 (MEDIUM), fixed as proposed.**
  `helperOrphanEnv` now takes `helperOrphanTier` or `helperOrphanBuild`, and the stand-in starts the orphan for the matching invocation.
  A new subtest runs the shared build with the orphan and a failing exit, and asserts exit 1, a failed build record without timeout, the orphan gone and the slots free.
  Commit `64636804`.
- **F3 (LOW), fixed as proposed, plus a twin.**
  `_lym/landing/summary.md` now states the real protection (Linux pins a group id while any member lives, the sweep finds nothing once it is empty, short of a pid-space wrap), and its Tests section describes both subtests.
  The same overstatement in the `KillTree` doc comment at `internal/proc/tied.go` (found by the fix-affected-set lens) now reads "a group that has emptied is left alone".
  The `_lym` file is not committed, per the round's rules; the `tied.go` comment is commit `proc: fix F3`.

Gate repair: the gate refused the `helperRoleStandIn` doc comment in `test_integration_test.go` because it wrapped one sentence over two lines.
The sentence is now on one line.

Gate: `lym gate test --wait --tier 1 ./internal/gatecli -- -run TestGateTest_Scenario` passed, 64 tests, including both new orphan subtests.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW, summary line omitted the pid-space-wrap window; fixed with its own change (the qualifier "short of a pid-space wrap" added to the summary).
- fix-affected-set: LOW, `internal/proc/tied.go` carried the same false start-time claim; fixed as proposed in the doc comment.

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

None.
