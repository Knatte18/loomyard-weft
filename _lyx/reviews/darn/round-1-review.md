---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/lenses/config_test.go:294
    summary: the pin on the operand sentence is dropped though the coverage lens still carries that check
    fix: In the pinned-sentence loop in TestConfigTemplate_DecodesAndIsSelfConsistent, pin "uses the operand the record names, not a near one" on the coverage lens as well as "A branch no planned row reaches is a finding.", for example by changing the map value to a slice of sentences and checking each.
    origin: lens:code-tests
---

### [LOW] Operand-sentence pin dropped from the pinned-sentence check

**Location:** `internal/lenses/config_test.go:294`

**Issue:** Decision D2 says each merged lens keeps the sentences `config_test.go` pins today where it carries the same check.
The old test pinned "uses the operand the record names, not a near one" on `criteria-coverage`.
The new `coverage` lens still contains that sentence, but the diff replaces the pin with the branch sentence only, so a later edit could drop the operand check without a failing test.
The summary says the pins moved to the new names, and this one did not.

## Rejected

- code-correctness LOW, the `coverage` text no longer says "a decision the record names that no row reaches": rejected, because the board entry's own draft text reads "a decision no card carries", the procedure still has each decision map to a row, and the "Your list" line asks for the card and the row or none.
- The summary's claims on lens order, emphasis line on each lens and the `_test` package detail in `forced-edits` all match the diff.
- No lens check is dropped from the merged texts: `affected-set`'s set method, the nil-seam check, `test-economy`'s procedure and the branch and criteria lists all appear in `coverage`, `row-validity` or `forced-edits`.
- No deleted lens name remains outside `docs/research/`, `PATTERN-test-economy` is an unrelated rule name, and the `affected-set` to `forced-edits` mapping in the perch-parity check names that one pair only.
- The `generic` row in `plan_test.go` exercises the same behavior as the `test-economy` row did, with the exemption removed.
