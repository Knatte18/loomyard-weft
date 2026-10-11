---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D5, Scope Out, Open risks)
    summary: The converged carry-over entry tells the plan writer each listed finding was addressed and its fix unseen, which is false for an every-fix-worse dispute, and the record keeps that text out of scope on the premise that it covers a dispute
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D4 Rationale, Auto-mode assumptions)
    summary: D4 is credited with making a persisting dispute settle and with giving the judge an answer to read, but the decision rule rules by class and severity alone and every settling path D5 names exists without D4
---

### [MEDIUM] The carry-over entry misstates an every-fix-worse dispute as a fix

**Location:** `_lyx/discussion/decision-record.md` D5 ("A closing-round dispute reaches `## Open risks` like any other addressed finding"), Scope Out ("The carry-over entry text in `internal/discussionparser/carryover.go`"), Open risks bullet 2; `_lyx/discussion/support-log.md` Rejected alternatives ("Change the carry-over text…").

**Issue:** `renderCarryOver` in `internal/discussionparser/carryover.go` renders a converged entry as "converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix", and `contracts/stencils/loom/loom-template-plan.md` tells the plan writer the entry "lists review findings whose fix in the record no fresh reviewer has seen".
For a false-premise dispute that reading is tolerable: the finding was answered.
For D2's new ground it is false: the fixer agrees the defect is real and leaves it in place, so there is no fix in the record, yet the entry tells the plan writer, whose sole input is the decision record, that one exists.
The record sees half of this (Open risks bullet 2: "An every-fix-worse dispute leaves a defect the fixer agrees is real"), but D5 and the rejected alternative keep the carry-over text unchanged on the premise that "the text 'addressed by the round's fixer' covers a dispute", which the new ground falsifies.
The fixer-report path is the only correct pointer, and the entry's own sentence tells the reader there is nothing to look for.
This is the converged-round bound the focus asks about: D2's bound states that the fixer's evidence is the only check in that round, and the one artifact that reaches the next producer then misdescribes the outcome.

**Fix:** Bring the converged closing string in `internal/discussionparser/carryover.go` into scope in D5 so it no longer asserts a fix, e.g. "each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which", with the matching sentence in `loom-template-plan.md` and `loom-template-rework.md` and their pinned claims or tests.
Move the "Change the carry-over text" item out of Rejected alternatives in `support-log.md`, and add an acceptance criterion for the new text.
If that is judged out of reach for this task, D5 must instead state the mislabel as an accepted risk in `## Open risks`, naming that the plan writer is told a fix exists where none does.

### [LOW] D4 is credited with a settling effect the decision rule does not give it

**Location:** `_lyx/discussion/decision-record.md` D4 Rationale ("the rule makes a re-raise carry an answer the judge and the next fixer can read"), Auto-mode assumptions bullet 3 ("stays open instead of settling"); `_lyx/discussion/support-log.md` Rejected alternatives ("persistent disputes stay open instead of settling") and Interview step 6.

**Issue:** `decisionRuleMarker` in `internal/shedadapters/bouncerprompt.go` decides from class, severity and whether the key was open earlier; the judge stencil says the judge never opens the artifacts and rules from the review's findings.
An answer written into a finding's `**Issue:**` changes none of those inputs, so it is nothing the judge can act on.
Every settling path D5 lists exists today without D4: the next reviewer drops the finding, the next fixer concedes and fixes, a design-class MEDIUM-or-worse key reaches CIRCLING from the checkpoint, a non-design BLOCKING key reaches the bounce-budget escalation, and any other key converges.
What D4 adds is information for the next fixer, which re-runs D1 on the re-raised finding and can now concede or re-dispute against the reviewer's answer.
The record and support log describe that as the difference between staying open and settling, which overstates it and names the judge as a reader that does not use it.

**Fix:** Restate D4's rationale and Auto-mode assumption 3 as: the reviewer's answer gives the next fixer the counter-evidence it needs to concede or to dispute again on new evidence, so a persisting dispute is decided on its merits rather than by repetition; drop "the judge" as a reader and "stays open instead of settling".
Align the support-log rejected alternative and Interview step 6 with the same wording.

## Focus verdicts

- **Settling paths per class (D4, D5, Auto-mode assumptions).**
  D5's three bullets match `decisionRuleMarker` for a dispute that persists into round 2 or later: a design-class MEDIUM-or-worse key on an earlier-open key rules `CONTINUE` and offers `CIRCLING` from the checkpoint; a BLOCKING key of any other class rules `CONTINUE` and has no CIRCLING path, so the bounce budget is its only escalation; every other key does not block.
  Each claimed settling path exists for its class; the overstatement around D4 is F2.
- **D2 and D3 bounds in a CONVERGED round, and the carry-over.**
  Both bounds state that no reviewer follows in the converging round and the fixer's cited evidence is the only check, which matches the burler round order (judge after fixer, no further review).
  D5 and Open risks describe `carriesOver` correctly: only open MEDIUM and BLOCKING entries reach `## Open risks`, and a LOW or NIT one stays in the fixer-report.
  The defect is in what the entry says about those findings, F1.
- **`Do not flag` findings and exactly one ground (D2).**
  Not a defect.
  The two grounds are mutually exclusive by construction: `premise false` requires the premise to fail and `every fix worse` requires it to hold.
  A `Do not flag` finding asserts a defect the fasit's rubric says is none, so the rubric line is fasit text that contradicts the premise, and D2 pins it to `premise false` explicitly.
  The fixer reads the same rubric through `burler-step-1-explore.md`, so it has the evidence it must cite.
