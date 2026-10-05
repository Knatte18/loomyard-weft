---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D11
    summary: D11's Decision omits the bound on the file-level parallel opt-out; it sits only in the Rationale
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:Acceptance criteria
    summary: The removed-phrase list misses phrases D3, D4 and D2 replace as the files carry them
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D5
    summary: D5's Webster-Review bullet says the finding names the covering test before listing a case that has none
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D1, D10, D11, D12, D13
    summary: D1, D10, D11 and D12 read as work to do, and D13 says the driver commits where Scope says committed
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D1
    summary: D1's last background bullet is absent from pattern/PATTERN-test-economy.md, though Acceptance says the file states every rule in D1
---

### [MEDIUM] D11's opt-out bound is not in the Decision

**Location:** `_lyx/discussion/decision-record.md`, D11.
**Issue:** The parallel opt-out is an escape hatch.
The Decision only quotes the PATTERN line, which says a comment at the test or once atop its file "names that state".
That a file-level comment excuses only tests touching the named state, and that a test touching none calls `t.Parallel`, appears only in the Rationale.
The rubric requires the Decision that introduces the escape hatch to state its bound.
**Fix:** Add a sentence to the Decision stating both clauses, and trim the Rationale to the reason.

### [MEDIUM] The removed-phrase list does not match the files

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.
**Issue:** Checked against the current files:
- `loom-plan-spec.md:299` says "independently reviewable and testable unit", which the list's "independently reviewable/testable unit" does not match; that form is the rubric's (line 44) and the claims table's.
- `loom-plan-spec.md:300` keys on "independent meaning or testability" and is not listed, though D3 makes granularity independent of testability.
- `loom-template-plan.md:71` carries "so it is testable" (the extract-a-helper example D2 removes) and the list omits it.
- `loom-plan-spec.md:403` carries "implementation-plus-its-own-new-test-file shape", which D4 replaces and the list omits.
- "independently testable" (symbol or reusable)" is a quoted phrase with an unquoted parenthetical.
- "normal shape" is a fragment of "the normal shape for such a card".
**Fix:** Rewrite the list with each phrase quoted as the file carries it, naming the file where the wording differs between files.

### [LOW] D5 names the covering test before a case with no covering test

**Location:** `_lyx/discussion/decision-record.md`, D5.
**Issue:** The Webster-Review bullet lists a duplicate, a per-helper test and a review-fix test for a non-gap, then says the finding "names the covering test".
The review-fix case has no covering test; a following sentence corrects that, so a reviewer reading the first sentence is misled.
Every other clause is grounded: the Plan-Review bullet names the existing or public-surface test, and the uncovered-behavior clause names the behavior.
**Fix:** State per clause what the finding names, in one sentence: the covering test for a duplicate, the public-surface test for a per-helper test, the non-gap finding for a review-fix test, the uncovered behavior for a missing one.

### [LOW] Already-done PATTERN work reads as a to-do in the Decisions

**Location:** `_lyx/discussion/decision-record.md`, D1, D10, D11, D12, D13.
**Issue:** Scope states the PATTERN work as done fact, but D1, D10, D11 and D12 open with "Add ..." or "The Testing section is ordered ...".
A plan writer reading a Decision without Scope could plan a card for it.
D13 says "the driver commits it before Discussion-Review", while Scope says it was committed.
**Fix:** Mark each of those Decisions as landed in the Discussion commit (D13) at the start of its body, and put D13's commit in the past tense.

### [LOW] D1 carries a rule the background file lacks

**Location:** `_lyx/discussion/decision-record.md`, D1.
**Issue:** D1's last bullet, "Out of scope of the rule: the coverage of existing tests", is not in `pattern/PATTERN-test-economy.md`.
Acceptance says that file states every rule in D1.
The content is already in Scope's Out list.
**Fix:** Drop the bullet from D1; Scope keeps the exclusion.
