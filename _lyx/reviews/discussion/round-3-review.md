---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md Decisions 2, 4 and 10
    summary: A Removable no candidate with no unique assertion and no foldable sibling has no permitted outcome
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Decisions 1, 2 and 4
    summary: Decision 4 lets a fold rename or absorb a receiving test or a core-named test that Decisions 1 and 2 protect only from deletion and rename
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 2 and Acceptance criteria
    summary: The receiver list is required only for deletions whose assertion moved, so no acceptance criterion can check it for completeness
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md Decision 12
    summary: The per-package coverage figure filters the profile by directory, which is ambiguous for the nested in-scope packages the Scope names
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 9
    summary: The no-tests clause covers a state deletion cannot produce and contradicts the splice bound that only replaces sections
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 5 and Acceptance criteria
    summary: The tier-move stay comment has no fixed spelling and no acceptance criterion checks it or its PR-body entry
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md added Scenario tests decision, Decision 11
    summary: A family is merged only when it cuts wall time, but the only measurement is the package-level before and after taken at the end, which cannot decide a single family and is noisy
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md added Scenario tests decision, Decision 8
    summary: Merging tests into ordered steps removes isolation with no bound on what a failed step lets later steps run on, and a step cannot be re-run alone
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md added Scenario tests decision, Decision 7, Acceptance criteria
    summary: The scenario decision is an unnumbered H3 in a different form, and Decision 7 and the acceptance criteria name folds and deletions but not merges
---

### [MEDIUM] A Removable no candidate can end with no permitted outcome

**Location:** `decision-record.md` Decision 2 (last-but-one sentence), Decision 4, Decision 10.
**Issue:** The tool lists `Removable: no` candidates in the same table as `yes` ones (`renderPackage` in `cmd/testtiming/redundancy.go` writes both), so the final rerun lists them.
Decision 2 forbids deleting one "on the report's evidence alone" and allows folding it.
Decision 10 requires a directive on every listed candidate but forbids annotating one with none of its three reason classes.
A `no` candidate that has no unique assertion and no sibling to fold with is neither deletable, foldable nor annotatable.
"On the report's evidence alone" also leaves open whether reading its assertions would permit deleting it.
**Fix:** State that a `Removable: no` candidate is never deleted, and give it exactly one of two outcomes: folded, or kept with a directive.
Add a fourth reason class to Decision 10 for it: the report marks it `Removable: no` and no fold applies, naming the covering test it would depend on.

### [MEDIUM] Fold escapes the protection Decisions 1 and 2 give a test

**Location:** `decision-record.md` Decision 1 (a test a core or testkit file names "is kept, not deleted or renamed"), Decision 2 ("A receiving test is never deleted later"), Decision 4.
**Issue:** A fold renames or absorbs its members.
Decision 4 says nothing about tests protected by Decisions 1 and 2, so a core-named test or a receiving test can be folded into a table under a new name.
That breaks the core reference, and it leaves the PR's receiver list and the `//testtiming:keep` reason naming a test that no longer exists.
**Fix:** In Decision 4, exclude from folding any test a core or testkit file names, and fold a receiving test only as the table's host, keeping its name.
Decision 2 says the same: a receiving test is never deleted, renamed or folded away.

### [MEDIUM] The receiver list cannot be checked for completeness

**Location:** `decision-record.md` Decision 2 Bound, third Acceptance criterion.
**Issue:** The PR lists a receiver "per deleted candidate whose assertion moved".
A deleted candidate with no unique assertion is not listed, so a reader cannot tell it from a deleted candidate whose unique assertion was dropped.
The acceptance criterion repeats the receiver requirement but names no check that can fail.
**Fix:** Require the PR to list every deleted candidate, each with either its receiving test or the words "no unique assertion".
Add to the criterion that every named receiver exists in the final tree.

### [MEDIUM] Coverage figure filter is ambiguous for nested packages

**Location:** `decision-record.md` Decision 12.
**Issue:** "Taken from the profile filtered by directory" does not say exact directory or prefix.
Scope names three in-scope packages nested under other directories (`internal/gitrepo/internal/gitoracle`, `internal/reedengine/render`, `internal/shuttleengine/claudeengine`).
A prefix filter counts the nested package's statements in its parent's figure.
The statement counting and the profile mode are also unspecified, while the rationale says "as the report measures them", and the tool uses `-covermode=set`.
**Fix:** Define the figure: statements covered over statements total, in the files directly in the package's own directory (exact match, no subdirectories), excluding `_test.go` files, from a `-covermode=set` profile.

### [LOW] No-tests clause in Decision 9

**Location:** `decision-record.md` Decision 9.
**Issue:** A candidate is deleted only when another test of the package covers it, so deletion always leaves one test.
Folding and tier moves keep tests too, so "a package left with no tests" has no way to occur.
The clause also adds a third splice action, removal, against the bound "replaces only in-scope sections and adds none".
**Fix:** Delete the clause, or if kept, state that the splice also removes a section and say why it is defensive.

### [LOW] Tier-move stay comment is free-form and unchecked

**Location:** `decision-record.md` Decision 5, Acceptance criteria.
**Issue:** The reason lives on the test, as a "one-line comment", so a later pass can find it.
Without a fixed spelling a later pass can only find it by reading the test, and no acceptance criterion checks the comment or the PR-body entry (the last criterion covers only Decision 1's references).
**Fix:** Fix a greppable spelling for the comment, such as `// tiermove: needs shared fake <name>`, and add an acceptance criterion that every integration test that stayed for a missing fake carries it and appears in the PR body.

### [MEDIUM] The scenario merge has no measurement that can decide a family

**Location:** `decision-record.md` "Added after Discussion" decision, Decision 11.
**Issue:** "A family is merged only when it cuts the package's wall time, which the before and after measurement shows."
Decision 11 measures per package, before on the base commit and after on the final tree, so it shows the total effect of every change in the package, not the effect of one merge.
The merge decision has to be made while editing, and the result is known only afterwards.
A family's gain can also be smaller than the package's load noise, which the Open risks name.
**Fix:** Measure each family alone, before and after the merge, with `go test -tags integration,tmux -run <the family's tests>`, median of three, and keep the merge only when the family is faster.
The PR lists each merged family with those two times.

### [MEDIUM] Scenario steps lose isolation with no stated bound

**Location:** `decision-record.md` "Added after Discussion" decision, Decision 8.
**Issue:** Separate tests each got a fresh fixture, so no test saw state another test left.
A scenario runs steps in order on one fixture: a failed step leaves state the later steps then run on and report misleading failures, and a step cannot be re-run alone with `-run`, which Decision 8 uses to tell a real order dependence from a tool artifact.
The decision requires a comment for a step that depends on earlier state, which does not bound either effect.
**Fix:** State the bounds in the decision: a failing step stops the scenario (`if !t.Run(...) { return }`), only tests that do not change the fixture's state in a way the others observe, or whose changes the comment names, are merged, and a step's comment names the step it depends on.
Decision 8's alone-rerun applies to the scenario test as a whole.

### [LOW] The scenario decision does not match the record's form or the neighbouring rules

**Location:** `decision-record.md` "Added after Discussion" decision, Decision 7, Acceptance criteria.
**Issue:** The decision is an unnumbered H3 with a provenance heading, and its Decision and Rationale sit in a different form from the numbered entries, so the rest of the record cannot cite it.
A merge renames and absorbs tests exactly as a fold does, so Decision 7 (references), Decision 1 (core-named tests) and Decision 2 (receiving tests) apply to it, but only folds and deletions are named there.
No acceptance criterion covers a merge.
**Fix:** Make it Decision 13 in the numbered form, with the provenance in a plain line.
Add merges to Decisions 1, 2 and 7, and add acceptance criteria: every merged family's before and after time is in the PR, and no merged step lost an assertion.

## Focus departures

None.
