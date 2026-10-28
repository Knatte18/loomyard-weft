# Fixer-report — code round 1

## Changes

- F1: in `internal/lenses/config_test.go`, `TestTemplateChecks_FlagBrokenTemplates`, the `writerFanGaps` subtest moved out of the `emphasisLineGaps` closure and is now a direct subtest; no assertion changed. Fixed as proposed. Commit `lenses: fix F1`.
- F2: in `TestConfigTemplate_DecodesAndIsSelfConsistent`, the tautological `board-scope` half of the `procedureLenses` assertion is dropped; the `leanness` check and its message remain. Fixed as proposed. Commit `lenses: fix F2`.

`lym gate test ./internal/lenses` is green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
