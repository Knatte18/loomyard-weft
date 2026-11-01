# Fixer-report, code review round 1

## Changes

- F1: `unmatchedLaunches` (`claudeengine/background.go`) takes `countPending`; `unfinishedForks` passes true, so a fork launch with no tool result yet is outstanding with an empty ID. `transcriptHasUnmatchedLaunch` keeps the old behaviour for the Stop-payload fallback. Added a `TestOutstandingForks` row.
- F2: `shuttlecli/cli.go` gains a `strandResolver` interface for the `reed` field, so the verb can be driven by a fake. Added `TestAwaitForksVerb` (no strand, unresolvable strand, no run record, ready, ready false with outstanding), a registration subtest and an `await-forks` stray-argument row in `TestRunCLI_PositionalArgValidation`.
- F3: added `TestRunnerAwaitForks` (capability refusal, unknown guid, reader receives the session id and pane cwd).
- F4: the three stencils bound the re-runs at five consecutive `ready: false` answers, then end the turn.
- F5: added `TestParseEvents_StopWithoutTaskListReadsBareForkLaunchAsWaiting`.
- F6: the `AwaitForks` error reads `find the run of strand %q: %w`.
- Self-check changes: the stencil bound is reworded to one reading ("until it has answered `ready: false` five times in a row"); the `background.go` header and `OutstandingForks` doc state that a never-recorded fork launch stays outstanding; the `await-forks` help says the caller bounds its re-runs.

- Gate repair: the `TestAwaitForksVerb` doc comment in `shuttlecli/cli_test.go` was one over-long sentence; it is now three sentences, one per line.

Fixed as proposed: F1 to F6. The commits group findings by file, not one per finding: F1 and F5 share `claudeengine`, F3 and F6 share `await.go`.

`lym gate test` is green for `shuttleengine`, `shuttlecli`, `burlerengine` and `twilldirective`.

## Self-check

- fix-consistency, LOW, `AwaitForks` error wording differs from its `session.go` twins: deferred, see below.
- fix-consistency, LOW, `await-forks` help text unbounded against the stencils: fixed (help says the caller bounds its re-runs; the number stays in the stencils).
- fix-consistency, LOW, `background.go` header and `OutstandingForks` doc omit the pending-launch case: fixed.
- fix-proposal, LOW, F1 pending launch stays outstanding forever if its result never arrives: fixed by documenting it; the stencil bound is the exit.
- fix-proposal, LOW, F4 count ambiguous: fixed, reworded in the three stencils.
- fix-code-correctness, LOW, same never-recorded launch as above: merged, fixed by documenting it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- The `session.go` wrappers (nine sites) and `enginedispatch_test.go:631` still say `is not a shuttle strand`. Making them match F6 changes the wording of other verbs and the test that asserts it, outside this review's findings. A parent or later round can decide.
