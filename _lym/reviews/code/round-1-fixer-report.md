# Fixer-report — code round 1

## Changes

- **F1** (fixed as proposed), commit `58dcc422`: `pattern/PATTERN-gate-slot.md:28` now names `discussionparser.Validate`, `CheckDecisionIDs`, the citation check `citationFindings` over `discussionparser.Citations`, and `CheckRelevant`.
- **F2** (fixed as proposed), commit `6547b04e`: `coversSeamTest` in `internal/loomshed/seamtest.go` matches the test's file path only against Uses and the Delete, Rename and Move groups, as D6 and spec check 55 say.
  A `stripHandles` helper is extracted from `strippedRefs`.
  The regression row `SeamTestEditingTheTestFileFails` in `internal/loomshed/gates_test.go` pins that an Edit naming the test's file does not cover it.
- `lym gate test ./internal/loomshed` passed before the commits (tier0: 324 passed, 0 failed).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the PATTERN line listed the checks in a different order than `ValidateLoomDiscussion` runs them; fixed as proposed by reordering, commit `ab4ba735`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
