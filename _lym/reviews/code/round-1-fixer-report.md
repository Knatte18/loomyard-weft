## Fixer-report, round 1

All nine findings are fixed.
Commits are grouped by package, not one per finding, because the findings share files (`dobbydeps.go`, `testmap_test.go`, the module docs) and `git add -p` is unavailable.
A later gate run flagged three comments, in `testmap.go`, `beamdown_test.go` and `orchcli/doc.go`, for a line over the length limit or a wrapped break.
They are now one sentence per line, with no change to their wording beyond the line breaks.
The tests passed before that repair: `lym gate test` tier 0 for `dobbyengine`, `orchcli` and `beamcli`, and tier 1 for `beamcli`, which holds the new `TestStartTestMap`.

### Changes

- **F1** (`b0f92e4c`), fixed as proposed.
  `startTestMap` no longer calls `runproc.EndProcess`.
  A `Register` failure logs one Warn naming the pid and the error, leaves the child running and returns nil.
  The doc comment, `internal/beamcli/doc.go`, `internal/dobbyengine/doc.go` and the `Deps.StartTestMap` comment state it.
- **F4** (`b0f92e4c`), fixed as proposed.
  `startTestMap` takes its binary lookup as a parameter, wired as `selfexec.LymBinary`.
  `TestStartTestMap` (`dobbytestmap_unix_integration_test.go`, tier 1, parallel) runs a shell stand-in as `sh gate testmap --max 3`.
  The test binary cannot stand in, since `tmuxkit.Main` refuses a positional first argument.
  A row asserts the command line, the working directory, one `runproc.RoleTestMap` record under `runproc.HubOwner` and its argv.
  A second row, with a run-proc directory below a regular file, asserts no error, no record and a child that still runs to its end (it would not if it had been ended).
- **F2** (`d1e3deac`), fixed as proposed.
  `tick` logs a `testMapStep` error through a new `testMapFailures` episode and goes on to `landBeam`, `announceLandings` and `Deps.Tick`.
  `testMapStep` still returns its wrapped errors.
  `TestRun_AFailingTestMapStartLeavesTheRestOfTheTick` shows the `Deps.Tick` hook reached on every tick and the start retried.
  `landBeam` itself is not reached by that test, because a beam slug makes the harness's worktree-gone check end the loop.
- **F3** (`0e60e7cf`), fixed as proposed.
  `orchCLI` has an `endTestMap` func, set in `wireCLI` only for the prime and nil for a beam.
  `beamDown` runs it after the dobby end on both the live and the not-live path and returns its failure wrapped.
  `endDobbyRoles` and its test are gone.
  `TestBeamDown_StopsOrchThenTheDobbyByRole` gains rows for a live dobby with a refresh, a dead dobby with a refresh and a failing refresh end.
  `orchcli/doc.go`, `beamcli/doc.go` and `docs/overview.md` say the refresh ends on a prime stop whether or not the dobby is live.
- **F5** (`d1e3deac`), fixed as proposed.
  `TestRun_NoPassWhileAMoveStands` has a `StartTestMap` fake and a count at the threshold.
  It asserts no refresh while the record stands and one once it is gone.
- **F6** (`d1e3deac`), fixed with my own change, per the self-check.
  The `dobbyengine/doc.go` Bound bullet and the `fabriccli/doc.go` sentence say the move counts a running refresh as a pass in flight and waits for it.
  They do not say the move ends it.
  The proposed bullet said it ends it, and that is false: the move record stands before the wait and stops new refreshes, so the refresh has finished by the time `EndDobbies` runs.
- **F7** (`d1e3deac`), fixed as proposed in its second form.
  `testMapStep` zeroes `LandingsSinceTestMap` while `test_map_every` is 0.
  The `countLandings` and `testMapStep` doc comments and `dobbyengine/doc.go` say so.
  The 0 row asserts the count stays 0 and that enabling the key later starts none.
- **F8** (`b0f92e4c`, `d1e3deac`), fixed as proposed.
  `TestHubDobbyDeps_PrimeTidyShuttle` asserts `TestMapRunning` and `StartTestMap` are filled for the prime and nil for the beam.
  `TestTestMapStep` has a row with a tidy agent in flight: fresh landings leave the count unchanged, and count once the agent ended.
- **F9** (`0e60e7cf`, `d1e3deac`), fixed as proposed.
  The stop test has the failing refresh end row.
  `TestTestMapStep` has a table with a failing `LoadConfig` and a failing `TestMapRunning`, each asserting the wrapped text, the kept count and no start.

## Self-check

- fix-consistency, LOW: the `Deps.StartTestMap` comment still said the process is registered without condition. Fixed.
- fix-consistency, LOW: `docs/overview.md` stop line did not mention the refresh end. Fixed.
- fix-proposal, MEDIUM: the F6 text said the move ends the refresh and that what it builds waits, which the wait rules out. Fixed with my own change, above.
- fix-proposal, LOW: an unregistered refresh has no bound beyond the verb's own timeouts, and an `UpdateState` failure after a start could start a duplicate. Not changed.
  This follows from the reviewer's F1 proposal, since ending an unregistered child needs `runproc.EndProcess`, which `PATTERN-child-processes` reserves for `lym tender reap`.
  The bound is the same one `dobbyengine/doc.go` already states for a dobby that dies.
  The review already rejected the `UpdateState` window.
- fix-proposal, LOW: `testMapFailures` recovers only when the step runs, so an episode open before a move closes after it. Not changed; harmless as the lens says.
- fix-code-correctness, LOW: the Register-failure row did not assert that the registration failed. Fixed: it asserts no record.
- fix-code-correctness, LOW: with the defect in place the F2 test hung until the harness timeout. Fixed: the start fake cancels the run on its second try, so the test fails at once.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
