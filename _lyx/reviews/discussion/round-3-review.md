---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7, and the Acceptance criteria line beginning "Darn-Bouncer accepts with such a finding open"
    summary: D7 rests on a false premise about the Bouncer, that CONTINUE and the circling escalation fire only on a design-class finding, so its acceptance line promises an unconditional accept the soft-cap rule does not give
    fix: In D7, replace the clause "the judge's CONTINUE rules and the circling escalation fire only on a gating finding, and a finding is gating only when its class is `design`, so a `decision`-class entry never holds the perch" with "a MEDIUM `decision`-class entry never makes the judge rule CONTINUE, so a judged CONVERGED round accepts with it open; in the soft-cap window, a round that another finding holds at CONTINUE or CIRCLING counts the open entry as design-wrong unless the judge tags it `mechanical`, so it can block that round's automatic accept and, open across a recorded continue, escalate with cause circling to the parent, the party who records or refuses the correction". In the Acceptance criteria, change "Darn-Bouncer accepts with such a finding open, without CONTINUE or a circling escalation, and its carry-over files it" to "Darn-Bouncer, judging a round whose only open entry at MEDIUM or worse is such a finding, rules CONVERGED and accepts it without CONTINUE or a circling escalation, and its carry-over files it".
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md, Interview item 6, the bullets beginning "scenarios MEDIUM + pattern-fit LOW" and "board-fit + leanness LOW"
    summary: The Interview says D5 drops the uncommitted-record check and a Constraint was reworded to stays recipe-neutral, but the current D5 keeps a committed-copy check, the Rejected alternatives reject relying on the post-append check alone, and no such Constraint exists
    fix: "In support-log.md's Interview item 6, end the bullet beginning \"scenarios MEDIUM + pattern-fit LOW\" with \"; later reversed: D5 refuses a record not yet committed or differing from its committed copy (see Rejected alternatives)\", and in the bullet beginning \"board-fit + leanness LOW\" change \"fixed, Constraint reworded to 'stays recipe-neutral'\" to \"fixed, the CLAUDE.md Constraint dropped\"."
---

### [MEDIUM] D7's Bouncer premise is false, and its acceptance line overstates the accept

**Location:** `_lyx/discussion/decision-record.md` D7, and the Acceptance criteria line beginning "Darn-Bouncer accepts with such a finding open".

**Issue:** D7 says the judge's CONTINUE rules and the circling escalation fire only on a gating finding, and that a finding is gating only when its class is `design`.
Neither half matches the Bouncer.
`internal/shedadapters/doc.go` says round 1 rules CONTINUE over any BLOCKING finding, whatever its class; D7 survives that only because it pins the entry at MEDIUM.
The soft-cap rule (`settleUndecided` and `softCapRule` in `internal/shedadapters/bouncer.go`) reads the judge's fix tag, not the class: `designWrongKeys` in `bouncerfiles.go` counts every open BLOCKING or MEDIUM entry not tagged `mechanical`, and a missing tag counts.
Darn's `review_max_bounces` defaults to 1 (`internal/darnengine/doc.go`), and Darn-Bouncer sets no `AutoAccept`, so the window opens from the second judged round.
In that window, when another finding holds a round at CONTINUE or CIRCLING, the never-closing parent-correction entry is a design-wrong key: it blocks the round's automatic accept, and while it stays open across a recorded continue it triggers the circling escalation earlier than the other findings alone would.
The outcome reaches the parent, the right party, so the design holds.
But the Decision's stated mechanism is false, and the acceptance line "accepts with such a finding open, without CONTINUE or a circling escalation" is unconditional, so a plan writer either writes a test the code fails in the window or changes the soft-cap rule to match the record.
On the focus question: the class and severity (`decision`, MEDIUM), the fixer's action (leave the code and defer, which `burler-step-fix.md` prescribes for a finding whose only sound change leaves the fixer's write surface) and carry-over (`carriesOver` takes every open MEDIUM entry) are each sound; only the Bouncer claim and its acceptance line need narrowing.

### [LOW] The support log's Interview contradicts the current D5 and Constraints

**Location:** `_lyx/discussion/support-log.md`, Interview item 6.

**Issue:** The bullet beginning "scenarios MEDIUM + pattern-fit LOW" records the fix as "D5 drops the uncommitted-record check and relies on the verb's existing post-append check".
The current D5 refuses a record not yet committed or whose working copy differs from its committed copy, and the support log's own Rejected alternatives reject "Rely on the post-append discussion check alone".
The bullet beginning "board-fit + leanness LOW" says a Constraint was "reworded to 'stays recipe-neutral'", and no Constraint in the decision record carries that wording or names `CLAUDE.md`.
Read as the record's current state, both bullets contradict the record; neither carries anything the plan writer needs, so the fix is a note in the support log only.

## Rejected

- Focus 2, PR-Align and an unrecorded parent correction (D9, D10): D9 states the route (escalate to the parent, never approve or reject, since only the parent can record or refuse), says the told value carries it, and the acceptance line "A darn PR-Align whose record's `## Open risks` holds a carried-over Darn-Review finding naming an unrecorded parent correction escalates to the parent" pins it. Not a defect.
- Focus 3, dropped-entry checks (D4, D5): `lyx fabric commit` commits the weft through `fabricengine`'s commit path, which takes the weft write lock (`internal/fabricengine/commit.go`) and has no amend, so a writer commit appends to the record's history and cannot move a reference that is the whole history. D5's post-commit check and its acceptance line ("`decision add` on a darn run refuses, naming its entry, when the committed record does not hold the entry after the verb's commit") match the Bound. Not a defect.
- Focus 4, the `pr-gate` scope-deviation definition (D10): `loom-template-alignment.md` line 52 holds the three-clause definition; D10 moves the whole definition into the told value, keeps the recorded-decision clause for loom only, and two acceptance lines pin both sides. Not a defect.
- D7's deferral as a bypassable revert guard: a writer can label any deviation an unrecorded parent correction, but the deviation then always reaches a parent turn through D9's escalation or a parent-held PR-Gate, which D7 and D9 state together. Not flagged.
