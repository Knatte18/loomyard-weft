# Round 1 fixer-report

## Changes

- F1 (LOW): fixed as proposed.
  Added `TestKeyListWayForward` to `internal/configreg/configreg_test.go`: a tier-0 parallel table test with rows add only, drop only, add and drop together, and order only, each asserting the exact returned string.
  Commit `1303c3eb`.
- F2 (NIT): fixed as proposed.
  Reworded the first doc-comment line of `keyListWayForward`.
  Commit `671e9400`.
  The self-check found the proposed wording ran over the 120-character line limit, so a follow-up commit split it at the comma before "so that" without changing the words.

`lym gate test ./internal/configreg`: tier0 passed, 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, F2 doc line over the line limit; fixed by splitting the line.
- fix-proposal: LOW, same line-length finding as fix-consistency (merged); fixed.
- fix-proposal: not a finding by the rubric, single-key rows only; the review asked for four rows and the omission does not worsen the fix, so no change.
- fix-affected-set: no findings.

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.
No fork returned no report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
