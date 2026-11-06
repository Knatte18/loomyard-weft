# Fixer-report, round 1 (code)

## Changes

- **F1** fixed as proposed (commit `2f036b1e`).
  In `TestCheckTexts` the five-phrase python loop is replaced by one assertion that the Codex python steer opens with `pythonRefusal`.
  The plain steer and the notice stay pinned byte for byte by `settings-golden.json`.
  The test's doc comment now names the Codex steer assertion.
  The reviewer thought `pythonRefusal` unreachable from the test; `rows_test.go` is `package shellguard`, so it is reachable and used directly.
- **F2** fixed as proposed (commit `fc9fdd9d`).
  The literal command entry is removed from `wants` in `TestConfigLong`, and the doc comment is restored to its earlier wording.
- **F3** fixed as proposed (commit `7a93d025`).
  `pythonRoutes` now reads "; for work that does not need Python itself, parse JSON with jq and text with grep or awk; ".
  The eight `denyPython=true` settings strings in `settings-golden.json` carry the new wording; the notices never carried it and are unchanged.

`lym gate test` over `./internal/shellguard`, `./internal/configcli` and `./internal/shuttleengine/claudeengine` passes: 914 passed, 0 failed.

## Self-check

All four forks returned a report; each is saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
