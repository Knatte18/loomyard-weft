# Fixer-report — merge, round 1

## Changes

- F1 (LOW), fixed as proposed: added a row to `TestLayerRunSettings` in `internal/configreg/runsettings_test.go`.
  The profile sets `open_ide_command` under `batten` to `x`, the task settings are empty, and the row expects a `*RunSettingsError` naming `run profile "light"`, `batten` and `open_ide_command`.
  Commit `f8a52d19e`.
- Gate repair: `TestBuild_EveryRegisteredEngineBuilds` failed for the `Command` engine, which had no entry in `engineMinimalConfig` and so built without its required `switch` and `command` keys.
  Added a `Command` entry in `internal/shedbuild/build_engines_test.go` naming `batten.open_window` and `batten.open_window_command`.
  `lyx gate test ./internal/shedbuild` and `./internal/configreg` are green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
  The forks ran before the gate repair, which is a one-entry fixture addition and was not re-checked by forks.

## Disputed

None.

## Deferred

None.
