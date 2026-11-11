# Round 2 fixer-report

## Changes

- F1 (MEDIUM, consistency): fixed as proposed in `_lym/landing/summary.md`.
  The "Own test files" paragraph now lists "an import the rule below does not allow" among the `MissTestFile` causes.
  The design bullet "Imports in a test file get no linked-package allowance" is replaced by the allowance rule `packageChanges` applies.
  An added import is allowed when it is the package's own import path, in the map's linked set, or in the standard library.
  A removed import is allowed when it is the package's own import path or in the standard library.
  Any other added or removed import gives `MissTestFile`, and a new test file importing `testing` selects its Test functions.
  No commit: the file lies under `_lym`, which this round never runs git against.

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

- F2 (decision-class), ruling from `ly:LOOM` notice #1225: the parent recorded the test-file import allowance as an `Added after Discussion` entry and asks to keep the code as it stands.
  Carried out: no code change, since `packageChanges` already holds the allowance.
  The F1 rewrite of `_lym/landing/summary.md` already states the same rule, so the summary needs no further change.
  Nothing left undone, and no flag to the parent.
