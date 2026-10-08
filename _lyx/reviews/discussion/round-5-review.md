---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:83
    summary: D5 makes every caller file of a renamed member a same-card target, but the spec defines Rename as an AST-aware rewrite of every reference with no ImpactSummary, and no decision reconciles the two
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:89
    summary: D5 says a package self glyph or file clears a same-package over-report, but D3 blocks a package self glyph beside a member of that package, and the same-card cover that Rename and re-sign require always lists the member
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:81
    summary: Caller coverage checks only Delete members, so a Delete by file path or package self glyph, which D6 and the spec sanction, skips it with no stated bound
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:73
    summary: The re-sign arrow needs a head that quarry.Name accepts, which excludes interface methods and struct fields, and D5 and D6 do not say what a planner writes for those members
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:86
    summary: The reference walk does not define its own package, so an external _test package in the same directory is matched by the bare-identifier rule and misses pkg.Name callers
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:143
    summary: The two-card re-sign criterion does not say whether the second card must again cover callers a first card already updated
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D3 bounds the missed file-versus-member pairing by a changed-files guard that covers both entries, but the guard is informational only, and the support log cites it as confining a card
---

### [MEDIUM] Caller coverage for a Rename Old side contradicts the spec's Rename mechanic

**Location:** `_lyx/discussion/decision-record.md:83`, `:144`, `:145`; `_lyx/discussion/support-log.md` Q21.
**Issue:** D5 requires every file that references a renamed member to be covered by a target of the same card, and justifies it with "a card that changes the declaration without its callers breaks the build".
`contracts/specs/loom-plan-spec.md` (Card types table, `Rename` row, and the paragraph "`Rename` requires no `ImpactSummary`") defines a Rename card as an AST-aware rewrite after which "every reference updates and the build passes".
The Quarry integration section repeats it: the rename primitive replaces the script, and the grep pass confirms zero remaining old-name references.
The premise "the card changes the declaration without its callers" is false for a Rename executed as specified, because the mechanic changes the callers itself.
D5 therefore forces the planner to enumerate every caller file of every renamed package-level member as an extra target, a burden the spec's no-blast-radius property for Rename exists to avoid.
Nothing in the record says whether the Rename pair itself counts as covering its callers (which makes the check vacuous for Rename), or whether the spec's Rename text is to change.
**Fix:** Decide in D5 which of the two holds and state it with its bound.
Either the Rename Old side is excluded from caller coverage because the mechanic and the build gate cover it, with the edit to the spec's Rename paragraph named in Scope;
or the same-card listing is kept as a plan-time declaration, with the spec's Rename text amended in the same commit and the burden stated in Open risks.
Align Q21, and acceptance criteria 144 and 145, with the choice.

### [MEDIUM] D5's clearing remedy collides with D3

**Location:** `_lyx/discussion/decision-record.md:89`, `:129`, `:138`, `:144`.
**Issue:** D5 says a same-package over-report on a package-level member is cleared by covering "that file or its package self glyph", and acceptance criterion 144 repeats "file, package self glyph or a member".
D3 makes a card listing a package self glyph and a member of that package a blocking redundancy finding (criterion 138), and a file plus a member that resolves into that file likewise.
A Rename Old side or re-signed member must be covered by the same card, and that card always lists the member itself, so for every same-package caller the package self glyph remedy trips D3.
The declaring file's path trips D3 too.
Only a member glyph whose span holds the reference, or a file path of another file of the package, can clear the finding in the same card.
The Open risk "force a planner to list a file it does not change" does not mention this.
The same holds for a `Delete` card that covers its own callers.
**Fix:** In D5, state which covers are reachable in the same card as the changing member (a file other than the declaring file, a member glyph) and that the package self glyph is reachable only on an earlier card, for a `Delete`.
Scope criterion 144 accordingly.
State whether references in the declaring file outside the member's own span count as callers, since "other than its own declaration" leaves it open.

### [MEDIUM] File and package Delete targets sit outside caller coverage with no bound

**Location:** `_lyx/discussion/decision-record.md:81`, `:96`.
**Issue:** D5 covers "every `Delete` member".
The spec's Card types table lets a `Delete` target "whole file(s)", and D6's stencil text directs a planner to a file path for "a whole file the card creates or deletes".
Deleting a file removes every member it declares, and deleting a package self glyph removes the package, yet callers of those members get no caller-coverage finding.
Existing `delete-before-reference` handles a package Delete only by import path and skips a file-path Delete.
The record introduces a new check and a stencil path that bypasses it, without saying so; the Bound lines list alias imports and files created by earlier cards but not this.
The same question applies to the Old side of a file Rename pair.
**Fix:** Decide in D5 whether file and package Delete targets, and file Rename Old sides, are in scope.
If out, add the bypass to the under-reporting Bound with what bounds it.
If in, say how the members of a file are found (the file self glyph's resolve answer carries a listing) and add an acceptance criterion.

### [MEDIUM] The re-sign arrow cannot be written for interface methods and struct fields

**Location:** `_lyx/discussion/decision-record.md:73`, `:77`, `:99`; `:35` (D1 shape classes).
**Issue:** D5 parses the head "like a Create head (one Go declaration, one symbol)" and matches it through `quarry.Name`.
`quarry.Name` takes Go declaration heads: `loom-plan-spec.md` says a populated interface is rejected as several declarations, and a bare field (`Name string`) or interface method (`Read() error`) is not a standalone Go declaration.
D1 lists struct fields and interface methods as member shape classes, and an interface method's signature change is among the highest-impact re-signs.
D5 says nothing about which member kinds accept the arrow; for these two the arrow is a parse-failure blocking finding, while D6's missing-arrow rubric finding would still ask the planner to add it whenever the Intent changes the member's parameters or results.
The two rules together are unsatisfiable for those members.
**Fix:** State in D5 the member kinds that take the arrow, and what a planner writes for an interface method or field re-sign (for instance, the enclosing type's head, or no arrow with the rubric finding exempted).
Carry the exemption into D6's second rubric finding.

### [MEDIUM] "Its own package" is undefined for external test packages

**Location:** `_lyx/discussion/decision-record.md:86`, `:89`.
**Issue:** The token rule matches a bare identifier "in its own package" and `<package clause name>` `.` `<Name>` "elsewhere".
Files with `package foo_test` sit in the same directory as the declaring package but are another package, and they reach the member as `foo.Name`.
If "own package" means same directory, those files are matched by the bare rule, skip every `.Name` occurrence (preceded by `.`), and miss real callers.
If it means same package clause, they fall under the qualified rule and are matched.
This repo holds hundreds of external test files, and D5 includes `_test.go` files explicitly, so the case is routine.
D1's shape class "package whose directory name differs from its package name" touches the same definition.
**Fix:** Define "own package" in D5 as the files of the declaring directory whose package clause equals the declaring file's, and say which rule an external test package in that directory gets.

### [LOW] Two-card re-sign leaves caller coverage of the second card open

**Location:** `_lyx/discussion/decision-record.md:143`.
**Issue:** Criterion 143 pins that the same member re-signed by two cards raises no `handle-canonical-collision` and no rewrite.
Caller coverage is name-based over the plan-time tree, so the callers card 1 updates still reference the name when card 2 is checked, and card 2 must cover them again.
The criterion says "validates clean" without saying whether the fixture's callers are listed on both cards, so a test can pass or fail the same implementation.
**Fix:** Add to criterion 143 that each card lists the caller files, or state that the second re-sign is held to the same-card rule against the plan-time tree.

### [LOW] The changed-files guard is informational and cannot bound or confine anything

**Location:** `_lyx/discussion/decision-record.md:58`; `_lyx/discussion/support-log.md`, Rejected alternatives, "The test card carrying the fixes".
**Issue:** D3's Bound calls the missed file-versus-member pairing harmless "since webster's changed-files guard covers both entries".
`internal/websterengine/doc.go` and `loom-plan-spec.md` state that the deviation list is "ALWAYS informational, never a failure condition".
The support log rejects a combined test-and-fix card because "webster's changed-files guard confines a card to its declared targets", which that guard does not do.
The D3 pairing is harmless because a redundant listing only adds plan noise, not because a guard bounds it.
**Fix:** In D3's Bound, replace the guard with the actual reason, or drop the clause.
In the support log, restate the rejected alternative on a premise that holds, for example that the Plan producer cannot foresee which defects the probe finds and a card's targets are fixed at plan time.
