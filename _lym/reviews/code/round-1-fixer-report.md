```yaml
round: 1
fixed: [F1, F2, F3, F4, F5]
```

## Changes

- **F1** (`internal/machinecli/set.go`): `sweepHub` gives a hub with no recorded entry and a non-empty `Lost` the new outcome `outcomeKept` ("kept by another owner").
  `setName` upgrades only `outcomeNothing` to `outcomePushed`, so a kept hub keeps its label.
  `runSet` still counts it unfinished through `len(hub.Lost) > 0`.
  New subtest `LostOnlyHubReportsKeptNotNothingToSweep` has the board ahead of origin and asserts the label and the non-zero exit.
- **F2** (`internal/machinecli/set_test.go`): new `TestName_Set` row `PrimeCollisionIsRefusedNamingHubAndPrime`.
  The shared refusal checks cover the unchanged machine file and zero owner writes.
- **F3** (`internal/machinecli/set_test.go`): new subtest `MachineFileWriteFailureIsRefusedNamingTheFileAndTheRerun`.
  **Fixed differently:** the review proposed a path under a regular file.
  That path fails at the read (`ENOTDIR` is not "not found"), so it reaches the read refusal and not the write refusal.
  The subtest instead uses a read-only directory that already holds the lock file, so the read finds no file and the write fails.
  It skips on Windows and as root.
- **F4** (`internal/boardengine/ownerset_integration_test.go`): new rows `Refusal_ReplaceSetWithSlugs`, `WithPrime`, `WithClaim` and `WithTakeover` assert the refusal naming the replace set and that the local and origin heads stay at the seed.
  **Fixed differently:** the review proposed rows in the `outcomes` table; that table's loop fails on any error, so the rows form their own table beside `refusals`.
- **F5** (`internal/boardengine/ownerset.go`): the `OwnerRequest` comment gains the sentence about the replace-set mode.

Commits: `4fdbe054` (F4, F5) and `19f3d080` (F1, F2, F3).
F1 to F3 share one commit because they edit the same two files.

## Gate repair

The verify gate failed tier 2 on `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why`.
The driver log named only `step  interrupted (loop-exited)`, since the loop's own account of why it stopped (`loop.detail`) was never read by the driver.
That account carries the poisoned status file's invalid state.
The failure comes from the merged `--until-stop` exit-recording change, not from the review's findings.
`sheddriver.Stop` now carries `Detail`, decoded from `loop.detail`, and the "handing the run back" log line records it (commit `395fbe3e`).
`lym gate test --tier 2 ./internal/loomcli` and `lym gate test ./internal/sheddriver` pass.

The verify gate then failed the same test again, with the loop's detail now in the log: `loop ... ended without an envelope; step  was in flight; loop exit: not recorded`.
The cause is in the test rig: `killDriversAndClearLog` kills a loop that may be mid-step, which leaves its pid file behind.
The next loop then refuses to step (`loopStartRefusal`), and the waiter reports the killed loop's stale record, so the poisoned state's refusal never reaches the log.
It reproduced with `-run TestSmokeBootstrapLifecycle -count=6` and above through `lym gate test --tier 2 ./internal/loomcli`.
`killDriversAndClearLog` in `internal/loomcli/smoke_test.go` now also removes the loop's pid file and envelope file (commit `8a9b9c19`).
The `-count=8`, `9` and `12` runs pass.

Gate: `lym gate test ./internal/machinecli` and `lym gate test --tier 1 ./internal/boardengine` pass.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
