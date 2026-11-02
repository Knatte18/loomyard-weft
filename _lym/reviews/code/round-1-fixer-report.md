# Fixer-report, code review round 1

## Changes

- **F1** fixed as proposed: `pattern/PATTERN-parent-directive.md` bullet 1 now names an interactive role's questions as the one place a question reaches a person, via the Go-filled `{{.mode_rules}}` marker, with the Discussion chair and the Drift-Verify seat under `verify_interactive` as the roles.
  The stale twin comment in `contracts/stencils/parentdirective_test.go` (found by the fix-affected-set lens) was reworded the same way.
- **F2** fixed as proposed: `internal/driftcli/wiring_test.go`, one table-driven `TestVerbStatus_OperatorSession` with an operator row and an autonomous row over `verbStatus`. Tier 1 passes.
- **F3** fixed as proposed: `runAgent` keeps the spawned table's `Interactive` and `prepare` passes it to `ResetRuns`, so `prepare` no longer reads `Config`.
  The fake `VerifyTable` in `verify_test.go` now sets `Interactive` from the config, so the operator-acceptance test still drives the mode.
- **F4** fixed as proposed: the `seats.go` header.
- **F5** fixed as proposed: the live-chair resume case in `verify_test.go` is now two rows; the second seeds an operator session while the config says autonomous, and both rows assert the mode at the resume and the run counter left untouched.
- **F6** fixed with my own change.
  `autonomousVerifyJudging` opens with the old sentence again.
  Not made: restoring `one autonomous session` in the stencil's opening line, since the stencil is shared with the operator mode, where it would be false.
  The remaining branch of the review's fix, correcting the claim in `_lym/landing/summary.md`, is outside my write surface; see Deferred.
- **F7** is `decision` class and went to the parent; left out of this report.

Tier 0 for `./internal/driftengine` and `./internal/driftcli` passed; tier 1 for `./internal/driftcli` passed.
One commit per finding, the comment fix in `contracts/stencils/parentdirective_test.go` is committed with the F1 follow-up.

## Gate repair

The verify gate failed on `internal/fabricengine` tier 1, which hit its 600s test timeout (`FAIL ... 600.662s`, parallel subtests waiting).
This round changes no file under `internal/fabricengine`, and the same package passed in 13.9s earlier in that log.
I re-ran `lym gate test ./internal/fabricengine --tier 1` alone: 1600 passed, 0 failed, wall 614.5s, so the suite is slow under load and the failure was a timeout.
No code change was needed; the timeout margin of that suite is the parent's to look at.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW: F6's stencil-opening branch cannot be carried out and the summary claim stays false; recorded under Deferred.
- fix-affected-set, LOW: stale twin comment in `contracts/stencils/parentdirective_test.go`; fixed as proposed.

## Disputed

None.

## Deferred, with reason

- **F6, summary claim:** the change description's claim that the autonomous text renders as before is still wrong for the stencil's opening line (`one autonomous session` became `one session`).
  Correcting it means editing `_lym/landing/summary.md`, which lies outside my write surface; the parent should reword the claim to say the opening line is reworded.
