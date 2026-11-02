## Changes

Every finding of round 1 is fixed. Commits are grouped by file, so findings that share files share a commit.

- **F1** (`8dab1222`): `internal/gateslot/tiers.go` declares `cardGateKey`; the three bare `"card"` key literals in `internal/gateslot/config.go` use it.
- **F2, F3** (`5705ac8d`, `internal/gatefixer/rowgate.go`): `waitOnly` and `waitThenRun` take `markOnPass`. The earlier entries pass false, so a passing earlier entry leaves the record live. The fixer entry's spawning path, its live-record branch and its `Final` pass true. `waitThenRun` returns the inner error before marking, so an erroring closure leaves the record live. Rows added to `TestWrapRowGate`: an earlier entry passes then verify fails (reply carried, `Expired` true between the entries, record terminal after, no second spawn), and an inner error leaves the record live.
- **F4, F5** (`69297718`, `internal/gatecli/detachfixer.go`): `runWithFixer` ends with the first result when the run's stderr file is absent or empty (`holdsFailureReport`). `fixAndRerun` marks the record terminal after the re-run returns, on every path that reaches the re-run. `TestRunWithFixer` gains a no-stderr row and an assertion that the record is live at the re-run.
- **F6** (`b6d7b26d`): `TestWriteNewRun_RecordsTheCallersRole` pins the role written to `request.json` for a fixer strand and a task strand.
- **F7** (`4344c76d`): `loomCLI.gotoGuardEndingFixers` builds the ending guard, with an `endFixers` field defaulting to `endRunFixers`. The three goto sites call it. `TestFixerEndingGuard` gains a row that runs the guard `specFor` builds with a recording end seam.
- **F8, F9** (`6b0c637f`): the unreachable `FixerTimeout() <= 0` branch and its test row are deleted; `darnGateEntry` names the Darn-Write key in `gateFixerBodies`.
- **Self-check fixes** (commit after `6b0c637f`): a pending earlier entry also leaves the record live; `internal/gatecli/doc.go` states the new mark order and the no-stderr exit; the no-stderr test asserts stderr is empty; a pending-entry row was added.

### Fixed differently

None, except F7's test scope, below.

## Disputed

None.

## Deferred

- **F7, partly:** the review asks for a test that runs the guard of each of the three goto sites. The `arm.go` site is tested through `specFor`. The `drivewiring.go` and `arbiterconsume.go` sites build their guard inside a driver-deps closure and a call into `shedarbiter.ConsumeAttempt`, which reach a real attempt directory or driver run; they go through the same `gotoGuardEndingFixers` helper and are covered only through it. Removing the helper call at one of those two sites is not caught by a test. A test for them needs a tier-1 hub fixture, outside this round's unit scope.

## Self-check

- fix-consistency, LOW: `internal/gatecli/doc.go` still stated the old mark order and the old exits. Fixed as proposed.
- fix-consistency, LOW: the `waitOnly` doc understated that a pending result also marked the record. Fixed by narrowing the skip to passed or pending, with the doc updated.
- fix-code-correctness, LOW: a pending earlier entry still marked the record terminal. Fixed (same change, with a pending row added).
- fix-code-correctness, LOW: the no-stderr test did not assert stderr is empty. Fixed with an assertion.
- fix-affected-set: no findings. It notes the F7 coverage limit, recorded under Deferred.
- fix-proposal: no findings.
