# Fixer-report, code review round 1

## Changes

Each fix is its own commit, except F1, F2 and F9 in one commit because they share files.

- **F1** (`internal/proc/memorycap_linux.go`): the scope runs `/bin/sh -c 'echo >&3; exec 3>&-; exec "$@"'` ahead of the command.
  `runMemoryCapped` reads the pipe after `Wait`.
  When the shell never wrote its line, the scope never ran the command, and the call returns an error that is no `*exec.ExitError`.
  A cancel returns its error as before.
  The new test `TestRunMemoryCapped_AScopeThatNeverStartsIsAnErrorAndNotTheCommandsExit` drives a starter of `/bin/false` through `RunMemoryCappedWithStarter` in `export_linux_test.go`.
  `RunMemoryCapped`'s doc names the replaced `cmd.ExtraFiles` and the scope-never-ran error.
- **F2** (`internal/proc/memorycap_linux.go`): the probe runs under `exec.CommandContext` with `probeTimeout` (5 s) and `probeWaitDelay`; a timeout returns `memoryScope{}`.
- **F4** (`internal/gatecli/testmap.go`, `testmappackage.go`, `redundantmutants.go`, `doc.go`, `docs/overview.md`): `testMapRequest` gets `cappedParallel`, set to `memoryCappedParallel` in production.
  `takeSlot` stores the clamped count in `testMapRun.sideBySide`, and `runBatch` and `runSideBySide` use it.
  The `lym gate testmap` help, the package doc and the overview sentence say fewer tests go side by side when memory is short.
  The help text dropped "Nothing else builds the maps:" to stay under the 2000-byte cap.
  `memoryCappedParallel`'s log message lost its "redundant" prefix, since testmap now logs through it.
- **F5** (`internal/testcover/testmain_test.go`, `runner_test.go`): a `TestRunAlone` row runs a stand-in that prints the runtime's out-of-memory line and exits 2, and wants `MemoryCapped`.
- **F6** (`internal/gatecli/redundancy_integration_test.go`, `testmap_integration_test.go`): the report step test gets a package `capped` whose new test `TestCappedHoards` keeps 384 MB under a 128 MB cap, and asserts it is listed not judged with `ended at the memory cap of 128 MB`.
  `redundancy_max_tests` rose from 9 to 10 in that test for the extra test.
  `TestGateTestMap` gets a last subtest that maps a `Hoards` test under the same cap and asserts a failed entry.
  Every `testMapRequest` literal in the tests sets `cappedParallel` through `unclampedParallel`.
  Both new tests ran green on a scope host in the tier 1 run.
- **F7** (`cmd/lym/spawnobservability_test.go`, `internal/proc/doc.go`): both reasons now say the one-time probe spawn goes unlogged and that a caller sees only its outcome.
- **F8** (`internal/gateslot/template.yaml`): the `redundancy_test_memory_mb` comment says the test is left out, recorded as failed by `lym gate testmap`, and a kill in `lym gate redundant`, and names both verbs in the side-by-side clause.
- **F9** (`internal/proc/memorycap.go`, `memorycap_windows.go`): the join window is stated in `RunMemoryCapped`'s doc and the Windows file's doc, the review's second option.
  A suspended start needs the main thread's handle, which `os/exec` does not expose, and no Windows host was available to test it.
- **F10** (no file): the review allows accepting the gap when no test drives `measurePackage`.
  None does: `cmd/testtiming`'s tests cover `judgePackage` and `renderPackage` only, and `measurePackage` builds a real coverage binary with the real `go`.
  The change description says the gap stays.

Tier 0 and tier 1 gates over `internal/proc`, `internal/testcover`, `internal/gatecli`, `internal/gateslot`, `cmd/lym` and `cmd/testtiming` passed after the F-fixes.
A tier 0 gate over `internal/gateslot`, `internal/proc` and `cmd/lym` passed after the self-check edits.

## Gate repair

The comment gate refused six comments added this round: two over-long lines and four wrapped breaks.
They are in `internal/gatecli/testmap.go`, `internal/gatecli/testmappackage.go`, `internal/gateslot/doc.go`, `internal/proc/memorycap_linux.go`, `internal/proc/memorycap_linux_test.go` and `internal/testcover/runner_test.go`.
Each is now one sentence per line, with no change of meaning.

## Disputed

Each entry is a self-check finding, ground `every fix worse`.

- **fix-proposal 1**, a probe that times out stays remembered by the `sync.OnceValue`, with the cause unlogged:
  a longer `probeTimeout` lengthens the stall F2 removes, and `proc` cannot log (`cmd/lym/spawnobservability_test.go` exempts `memorycap_linux.go` for the import cycle).
  `memoryCappedParallel` already logs `cap_covers_tree` and the headroom.
- **fix-proposal 2**, a scope that fails to start in a run's own `cmd.Env` is an error on every run, with no fallback to the data segment limit:
  a fallback would run the command as F1 asks the call not to, uncapped by the kernel and unreported.
  F1 requires the error.

## Deferred

Nothing deferred.

## Rulings

- **F3**: the parent ruled that the fallback keeps `RLIMIT_DATA` in place of `RLIMIT_AS`, as the code already does.
  The ruling is redundant: `runDataLimited` already calls `unix.Prlimit` with `RLIMIT_DATA`, so I changed no file for it.
  I told my parent once with `lym creel post --parent`.

## Self-check

- fix-code-correctness: returned a report, no findings.
- fix-consistency: LOW, `internal/gateslot/doc.go` did not name testmap's clamp. Fixed as proposed by adding the sentence.
- fix-proposal: LOW, probe timeout remembered for the process. Disputed, every fix worse.
- fix-proposal: LOW, per-run scope start failure never feeds back into the probe. Disputed, every fix worse.
- fix-proposal: LOW, the F7 reason claimed a logged outcome that `internal/testcover` and `cmd/testtiming` never log. Fixed: both reasons now say a caller sees only the outcome.
- fix-affected-set: returned a report, no findings; it left F3, F9's Windows path and F10 unchecked.
