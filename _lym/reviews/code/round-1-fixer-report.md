# Fixer report — code review, round 1

## Changes

Each fix is its own commit; the tier-0 gate over the five touched packages and the tier-1 gate over `beamcli` and `fabriccli` pass.

- **F1** (`81c8647a`): `startTestMap` in `internal/beamcli/dobbydeps.go` now sends the pass's stdout and stderr to a file made with `lymtemp.CreateTemp`, not a `tailBuffer` pipe.
  The dobby closes its copy of the descriptor once the child starts, reads the file's last `passErrorTailBytes` after the pass exits through the new `readFileTail`, and removes the file.
  A dead dobby can no longer break the pass's writes.
  The stand-in script in `dobbytestmap_unix_integration_test.go` records whether its stdout is a pipe, and the test asserts it is not.
- **F2** (`c42371be`): the in-flight scan of `dobbyPassInFlight` moved to `liveUnsparedChild` in `internal/fabriccli/anchordeps.go`, behaviour unchanged.
  `anchorchild_linux_test.go` drives it through `LiveUnsparedChildForTest` with a real child of a stand-in shell: a child registered as the upkeep pass, a push or a board sync is spared, and a child registered under another role or nowhere counts.
- **F3** (`9ede7517`): `internal/orchcli/doc.go`, `stop.go` (comments and the error text), `cli.go` and `beamdown_test.go` now call the ended process the coverage-map upkeep pass, which registers itself; the asserted substring changed with the error text.
- **F4** (`327fc8d5`): the `ReadState` subtest in `internal/dobbyengine/state_test.go` now writes a record holding `landings_since_test_map`, reads it before the update, and checks the key survives the rewrite.
- **F5** (`1fb547f0`): a `TestReconcileAll` row in `internal/configsync/configsync_test.go` seeds a `beam.yaml` holding `test_map_every: 0` and asserts the apply migrates it to `test_map_upkeep: false` and leaves no `test_map_every`, which pins Migrate before the retired-key prune.

- **Gate repair:** the gate refused two comments, so I split the stand-in script's header in `dobbytestmap_unix_integration_test.go` and the `liveUnsparedChild` doc in `anchordeps.go` into shorter sentences, one per line.
  Neither edit changes behaviour.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, MEDIUM, F1 output file lives in a dobby-scoped `lymtemp` run directory (Windows removal): disputed, premise false.
- fix-proposal, LOW, F1 output file never removed if the dobby dies: disputed, premise false.
- fix-proposal, LOW, F1 test only proves stdout is not a pipe: disputed, premise false.
- fix-proposal, LOW, F5 row does not reach the hub-wide walk: disputed, premise false.

## Disputed

- **fix-proposal MEDIUM (F1, Windows run-directory removal)** — premise false.
  The consequence it names is an error from `lymtemp.Remove`, and `cmd/lym/main.go:64` discards that error with `_ = lymtemp.Remove()`.
  The directory then stays until its owner is dead and the sweep removes it (`internal/lymtemp/sweep.go`, `ownerDead` and `removeRunDirectory`).
  The pass's writes go to an open file, so nothing breaks either way.
- **fix-proposal LOW (F1, file left behind when the dobby dies)** — premise false.
  The file sits in the dead dobby's `lymtemp` run directory, which the sweep removes once its owner holds no lock, so nothing leaks.
  The pass's output volume is the pass's own, and the file is read only through its last `passErrorTailBytes`.
- **fix-proposal LOW (F1, test proves only that stdout is not a pipe)** — premise false.
  The defect was a pipe whose read end only the dobby held.
  With stdout a file there is no read end to lose, and the check fails against the old `tailBuffer` pipe, as `[ -p /dev/stdout ]` is true there.
- **fix-proposal LOW (F5, no hub-wide board-dir case)** — premise false.
  `beam` is not a `HubWide` module (`internal/configreg/configreg.go:186`), so `ReconcileHubWideAt` never walks its file.
  Its reconcile walk is `ReconcileAll`, which the new row drives.

## Deferred with reason

None.

## Rulings

None.
