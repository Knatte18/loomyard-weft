# Fixer-report, darn round 1

## Changes

- F1, fixed as proposed (commit `d0e491963`): in `internal/lenses/config_test.go`, the pinned-sentence map of `TestConfigTemplate_DecodesAndIsSelfConsistent` now holds a slice of sentences per lens.
  The `coverage` lens pins "uses the operand the record names, not a near one" beside the branch sentence.
  `lyx gate test ./internal/lenses` passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
