---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D5
    summary: D5's Plan-Review bullet introduces the redundant-test finding edge without the naming-the-covering-test bound that D1 requires
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D11
    summary: D11's parallel opt-out does not state that a file-level comment is valid only when every test in the file touches the named state
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Acceptance criteria
    summary: Acceptance criteria list the phrases that must disappear but omit two that D3 and D9 also remove, and D3 and D9 spell the granularity phrase differently
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Scope, D8, Constraints
    summary: The D13 split is not carried through Scope, D8 and Constraints, which still read as instructions for work already committed
---

### [MEDIUM] D5 Plan-Review bullet has no stated bound

**Location:** `decision-record.md`, D5.
**Issue:** D1 bounds the new BLOCKING edge by requiring the finding to name the covering test.
D5 repeats that bound for the Webster-Review bullet only ("is a finding, which names the covering test").
The Plan-Review bullet reads "a card that plans a new test function or file for behavior an existing test already covers, or one test per symbol, is a finding" and names nothing.
The "one test per symbol" clause has no covering test to name at all, so a Plan-Review reviewer can block on it without any grounding.
The same Decision also lets the Webster bullet's third and fourth clauses (a review-fix test for a non-coverage-gap finding, and a behavior with no coverage) escape the bound, since neither has a covering test to name.
D5's own sentence says the bullets restate "none of it beyond the finding condition", so the bound must be part of the finding condition.
**Fix:** In D5, state per clause what grounds the finding: a duplicate names the existing covering test, and a per-symbol card names the public-surface test that covers the symbol.
For the clauses with no covering test, state what the reviewer cites instead (the fix's finding, the uncovered behavior).

### [LOW] D11 opt-out does not bound a file-level comment

**Location:** `decision-record.md`, D11.
**Issue:** The opt-out lets a test run serially when a comment names process-global state.
D11's Rationale says a file-level comment "covers a file whose tests all share one fixture", but the Decision does not say that is the condition.
As worded, a comment atop a file excuses every test in it, including one that touches no named state.
**Fix:** In D11's Decision or Rationale, state that a file-level comment excuses only tests that touch the named state, and that a test touching none calls `t.Parallel`.

### [LOW] Phrases to remove are listed incompletely and spelled two ways

**Location:** `decision-record.md`, Acceptance criteria against D3, D9 and Notes.
**Issue:** The second Acceptance bullet requires that "Bundles its own test", "independently testable or reusable" and "normal shape" for a new test file be gone.
D9 also replaces the claims on "not a substitute for a bundled test" and "independently reviewable/testable unit", and Notes names "the bundle-your-own-test rule" in Step 0.
None of those three is in the bullet, so a plan can pass the criterion and leave them behind.
D3 quotes the new wording as "independently reviewable unit" and the old as "independently reviewable and testable unit", while the pinned claim reads "independently reviewable/testable unit".
**Fix:** Extend the Acceptance bullet to the full set of removed phrases, and quote the old granularity phrase in D3 and D9 in the form the file actually carries.

### [LOW] D13's split is not carried through Scope, D8 and Constraints

**Location:** `decision-record.md`, Scope first bullet, D8, Constraints.
**Issue:** D13 says the PATTERN half is done and no card re-creates it.
Scope's first "In" bullet and the audit bullet still list the PATTERN revision and the `PATTERN-refusal-way-forward` rewording as work, with a trailing note.
D8's Decision is phrased as instructions ("Reword `PATTERN-refusal-way-forward`'s overview line to ...") for files D13 says are committed, and its closing sentence tells Plan the audit "applies the same reading to every other PATTERN entry", which invites a re-audit although the audit is finished.
Constraints still carry items that bind only the finished half: the `internal/pattern` overview-format check on "the new line", and the `docs/` link-integrity bullet, though no change here adds a link under `docs/`.
A plan writer reading Scope or D8 first can reasonably plan a card for the PATTERN half.
**Fix:** Move the already-committed PATTERN work out of Scope's In list into a single "Already done (D13)" line, split D8 into the done wording (stated as fact) and the `refusal-spec.md` alignment that is Plan's, and drop or reframe the two moot Constraints.
Keep the CLAUDE.md departure bullet only if the plan writer needs it, since D13 already states it.

## Judged and not a defect

- **Support-log versus decision-record requirements.**
  Every rejected alternative's constraint is carried in the record: the worked-example and golden-fixture rule (Scope Out, Constraints), the reaching-test obligation (D8), the severity rule (Auto-mode assumptions) and the no-new-test rule (D9, Acceptance criteria).
  The Q10 reasoning is deliberation narrative; its one requirement, that the PATTERN commit precede Plan, is already in D13.
  Nothing the plan writer needs lives only in the support log.
- **D9's no-new-test claim.**
  It holds: `claims_test.go` is a table of `wantAll`/`must` rows, and the old phrases sit in rows (lines 113 and 149), so replacing them in place adds no `Test` function.
  The new D4 to D7 wordings stay unpinned, which is consistent with D9 and is not required.
- **Acceptance criteria restating D1, D10, D11 and D12.**
  The first bullet points at those Decisions rather than quoting them, and serves as the keep-true check on done work, so it is not a duplicate.
- **D1's bound.**
  D1 states what the new BLOCKING edge lets through (a finding on a named duplicate) and its bound; the gap is only in D5 (F1).

