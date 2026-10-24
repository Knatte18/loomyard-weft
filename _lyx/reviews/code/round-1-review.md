---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/planglyph/donecheck_resolve_test.go:63
    summary: The Rename new side counterpart path in DoneChecks has no test row.
    fix: In TestDoneChecks_Verdicts add a row with a Rename group whose new side is plan:sub#TestFoo and whose member is declared only in sub_test (a_test.go with package sub_test), expecting no finding. Add it as a row, not as a new test function.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/planglyph/handle_test.go:608
    summary: No row pins that an exact delta match wins over the counterpart, or that a counterpart match still counts toward bind-count-mismatch.
    fix: Extend TestBindHandles_HandleBindsToTheOtherPackageOfItsDirectory or the existing partial-match test with rows for a delta holding both sub#TestNew and sub_test#TestNew (expect the exact glyph), and for a card with one counterpart-matched handle plus one unmatched handle (expect bind-count-mismatch and no rewrite).
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/planglyph/handle.go:481
    summary: Two handles of one card, sub#TestX and sub_test#TestX, can both bind to a single created sub_test#TestX, so one delta symbol is bound twice and bind-count-mismatch is missed.
    fix: In BindHandles track the glyph ids already claimed within a card and refuse a counterpart bind whose glyph is already claimed, so the second handle counts as unmatched.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/planparser/doc.go:20
    summary: planparser/doc.go lines 20 and 82, and contracts/specs/loom-plan-spec.md line 198, still call goPackageDir the one sanctioned Go unit adjustment, which GoPackageCounterpart now contradicts.
    fix: Reword doc.go lines 20 and 82 and the spec sentence at loom-plan-spec.md:198 to name goPackageDir and GoPackageCounterpart as the sanctioned adjustments, matching pattern/PATTERN-glyph-conversion-chokepoint.md.
---

### [MEDIUM] Rename new side counterpart is untested

**Location:** internal/planglyph/donecheck_resolve_test.go:63
**Issue:** The diff extends the counterpart match to `rename-not-done-new`, as the decision record's auto-mode assumption asks.
Only Create rows were added, so removing the Rename half of the condition breaks no test.

### [LOW] Binding precedence and mismatch via counterpart are unpinned

**Location:** internal/planglyph/handle_test.go:608
**Issue:** The new test covers only the all-matched case.
Exact-over-counterpart preference and a counterpart match counting toward `bind-count-mismatch` have no row.

### [LOW] A delta symbol can be double-bound through the counterpart

**Location:** internal/planglyph/handle.go:481
**Issue:** `boundGlyph` checks each handle independently.
A card owning both package spellings of one member binds both to the single created symbol, which the focus directive asks to rule out.
The scenario is unlikely, since `handle-collision` covers only identical spellings.

### [LOW] Stale "one sanctioned adjustment" statements

**Location:** internal/planparser/doc.go:20
**Issue:** `doc.go` lines 20 and 82 and `contracts/specs/loom-plan-spec.md:198` describe `goPackageDir` as the sole sanctioned Go unit adjustment.
The diff updated the background file and the `goPackageDir` comment but not these.

## Rejected

- Counterpart accepts a member that existed before the card ran (lens:code-correctness): the decision record's Bound accepts any declaration of the same member name in the other package.
- No `DoneChecks`-level test for a C# plan (lens:code-tests): the helper's csharp row pins the gate, and a second test would be redundant under test economy.
- Unit-only `_test` input yielding an empty unit (lens:code-correctness): unreachable from a valid glyph.
