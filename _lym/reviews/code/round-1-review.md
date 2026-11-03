---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/planparser/renumber_test.go:46
    summary: The kept-range branch of mapsFaithfully on insert (a range the inserted card falls strictly inside, or one ending at the insert position) has no test row, so deleting the clause would fail nothing.
    fix: In internal/planparser/renumber_test.go add a prose line to the TestRenumber fixture and the insert rows' expectations. Use a range such as cards 1–3 with a card inserted at 2, expected to stay whole and be listed in the mentions. Add a boundary row such as cards 1–2 with an insert at 3, expected to stay unchanged and not be listed. Fold both into the existing wantProse and wantMentions fields rather than adding a new test function.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/planparser/renumber_test.go:392
    summary: A card that keeps its number and changes only by a bare reference is journaled and restored on a failed write, but no test faults its write; the existing failure test faults a card that also changes by card id.
    fix: In internal/planparser/renumber_test.go add one row to the existing failure-reversal test, or an assertion in it, that faults the write of a keeps-its-number card whose only change is a bare card N reference. Assert the directory is restored byte for byte and no journal is left.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/planparser/renumber_test.go:46
    summary: No test pins that a bare card N on a Card Index line or on a card heading line is left alone, so a regression that rewrote either would pass.
    fix: In internal/planparser/renumber_test.go give the TestRenumber fixture an index line and a heading carrying a bare reference to a moved card, and assert in the existing wantProse and overview checks that only the Card Index renumbering touches them.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/planglyph/index_test.go:127
    summary: glyphlessFile's failure paths (toc error, lossy file, error entry, nil Symbols, file absent from the toc) have no row, so the refuse-only guarantee the summary claims is pinned only incidentally by the empty-directory rows.
    fix: In internal/planglyph/index_test.go add rows to TestIndex_AppendsPlanChecks over a fixture repository for a test file that does not exist and for a test file with a syntax error, both expected to keep the one edit-test-file finding. Add a row where a same-base file in another directory has symbols to pin the path.Base match.
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-plan-spec.md:226
    summary: The rewritten sentence says they leave it as is and report its line, and a card id never needs that, so the antecedent of that is unclear and the closing clause no longer follows.
    fix: In contracts/specs/loom-plan-spec.md reword the sentence at line 226 as two sentences. State that the verbs leave a bare reference naming the removed card, or a range the inserted or removed card falls inside, as it is and report its line. Then state that a card refers to another card by its card id, which both verbs always rewrite.
---

### [MEDIUM] Kept-range branch on insert is untested

**Location:** internal/planparser/renumber_test.go:46, internal/planparser/renumber.go:729
**Issue:** The commit adds the rule that a range the inserted card falls inside is left whole and reported.
The only kept-range row is the remove case where the range starts at the removed card, which the `number == e.position` clause catches.
The `inRange && previous < e.position && e.position <= number` clause is reached by no row, and neither are the hyphen, em dash and `and` list forms of `cardMentionRe`.
I traced the clause by hand with the correctness fork and it is right, so this is a coverage gap and not a defect.

### [LOW] Bare-only change to a card that keeps its number is not fault-tested

**Location:** internal/planparser/renumber_test.go:392
**Issue:** `applyRenumber` journals and writes such a card only when `len(refs) > 0`.
The CLI insert row pins the successful write, but the failure test faults `08-alpha.md`, which changes by card id too.

### [LOW] Skipped heading and Card Index lines are untested

**Location:** internal/planparser/renumber_test.go:46
**Issue:** `cardIndexLines` and `headingLine` exist to keep these lines out of the bare scan, as the removed scan did.
No row has a bare reference on such a line.

### [LOW] glyphlessFile failure paths have no row

**Location:** internal/planglyph/index_test.go:127
**Issue:** The new rows pin the empty-symbols admit and the member refusal.
The toc-error, lossy, error-entry, nil-Symbols and absent-file returns, and the `path.Base` match, are covered only by the unchanged empty-directory rows, which reach the toc error alone.

### [NIT] Ambiguous sentence in the renumber spec

**Location:** contracts/specs/loom-plan-spec.md:226
**Issue:** After the rewrite, `a card id never needs that` has no clear antecedent.

## Rejected

- Remove-card rewriting bare references is outside the Goal's wording (rejected as a scope finding). Both verbs share `applyRenumber`, the old behavior reported mentions for both, the Goal's expected outcome is about renumbering references, and the change description states it.
- The `--help` text imprecise at the range-end boundary (rejected, lens:code-affected-set). A range ending at the insert position has the inserted card inside it, so `falls inside` is accurate.
- A zero-padded bare number such as card 09 loses its padding (observation of lens:code-correctness, not a defect). Bare prose does not carry padding.
- Nothing in the verify gate's subjects was re-derived: build, test and tier purity are the gate's.
