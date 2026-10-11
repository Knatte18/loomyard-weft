---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:79
    summary: D5 and D4 claim the CIRCLING path settles a persisting dispute, but CIRCLING only fires on a gating (design-class) finding, so a persisting dispute on any other class never reaches it
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:50
    summary: The D2 and D3 bounds depend on a next fresh reviewer that the closing round never gets, and Open risks wrongly says a carry-over entry always points at a closing-round dispute, which is false for LOW and NIT findings
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:46
    summary: D2 lists the rubric's Do-not-flag list as evidence for the every-fix-worse ground, while Auto-mode assumptions classify a Do-not-flag finding as a false premise, so one finding gets two grounds
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:45
    summary: D2's every-change test does not say whether changes outside the fixer's write surface count, so a finding whose only sound change leaves the surface is either disputed (D2) or deferred (D3)
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:70
    summary: D4 asks the reviewer to answer a dispute's evidence in the Issue line but does not say when, and the clean-room rule makes the reviewer write its findings before it may read any fixer-report
---

### [MEDIUM] D5's CIRCLING premise holds only for design-class findings

**Location:** `_lyx/discussion/decision-record.md:79`, and the same premise at `:106` (Auto-mode assumptions) and in D4's rationale (`:73`).

**Issue:** D5 says "the existing CIRCLING path settles a dispute that persists", and D4 and the Auto-mode assumptions justify D4 by saying that without it the broader dispute ground "feeds the CIRCLING path".
In `internal/shedadapters/bouncerprompt.go` (`decisionRuleMarker`), CIRCLING is earned only by "a gating finding", and "a finding is gating when its class is `design`".
A dispute on a `scope`, `decision` or `consistency` finding that a later reviewer re-raises therefore takes one of two other paths.
At BLOCKING it rules `CONTINUE` round after round and escalates with cause budget once `review_max_bounces` is spent.
At MEDIUM or below, from round 2 on, it does not block, so the segment converges, carrying the finding only if it is MEDIUM.
D5's claim that the judge and convergence need no change rests on the CIRCLING premise, so the plan writer and the docs D6 edits inherit a false account of how a dispute settles.

**Fix:** Restate D5's second sentence as the path the decision rule takes.
A persisting dispute on a design-class finding is settled by CIRCLING from the checkpoint round on.
A persisting BLOCKING dispute of any other class runs `CONTINUE` until the bounce budget escalates it.
A persisting non-BLOCKING dispute of any other class converges from round 2 on, and the carry-over records it only when it is MEDIUM.
Then reword D4's rationale and the Auto-mode assumption so they no longer cite CIRCLING as the only settling path.

### [MEDIUM] The D2 and D3 bounds have no check in the closing round, and LOW and NIT disputes leave no carry-over pointer

**Location:** `_lyx/discussion/decision-record.md:50` (D2 Bound), `:63` (D3 Bound), `:80` (D5), `:112` (Open risks).

**Issue:** The focus asks whether D2's bound can be enforced by anything other than the fixer's own judgment.
The decision record names only the next fresh reviewer as that check (D3's Bound says so outright, and D2 relies on D5).
The round the judge rules CONVERGED has no next reviewer.
That includes round 1 whenever its review holds no BLOCKING finding and no design-class finding at MEDIUM or worse.
So in that round the every-fix-worse dispute and the fixed-differently change are bounded only by the fixer's own cited evidence.
Open risks says this is covered "as today" because "the carry-over entry's fixer-report path is the only pointer to it".
That is false for LOW and NIT findings: `carriesOver` in `internal/shedadapters/bouncercarryover.go` keeps only BLOCKING and MEDIUM ledger entries, so a closing-round LOW or NIT dispute gets no carry-over entry at all.
D5's "a closing-round dispute reaches `## Open risks` like any other addressed finding" has the same gap.
The new ground is what makes this matter: today's false-premise dispute leaves no real defect behind, while an every-fix-worse dispute leaves a defect the fixer agrees is real.

**Fix:** In D2's Bound, state that in the round the segment converges the only bound is the fixer's cited evidence, since no reviewer reads it.
State the same in D3's Bound.
In D5 and the Open risks entry, say that only MEDIUM and BLOCKING closing-round disputes reach `## Open risks`, and that a LOW or NIT one is recorded only in that round's fixer-report.

### [MEDIUM] A Do-not-flag finding has two dispute grounds

**Location:** `_lyx/discussion/decision-record.md:46` (D2 evidence list) against `:105` (Auto-mode assumptions).

**Issue:** D2 is the ground for "a finding whose premise holds", and its evidence list includes "the rubric (its `Do not flag` list included)".
The Auto-mode assumption says "a finding the rubric's `Do not flag` list excludes counts as a false premise".
So a fixer facing a Do-not-flag finding can name either `premise false` or `every fix worse`, and D2 requires each `Disputed` entry to name exactly one ground.
The acceptance criteria copy D2's evidence kinds into `burler-step-3-fix.md`, so the contradiction ships in the stencil.
D2's clause "disagreement with the rubric ... never justif[ies] a dispute" does not conflict: citing the Do-not-flag list applies the rubric rather than disagreeing with it.
The classification of a Do-not-flag finding is also a rule the stencil must state, so it belongs in D2 rather than only in Auto-mode assumptions.

**Fix:** Drop "(its `Do not flag` list included)" from D2's every-fix-worse evidence list.
Add one sentence to D2 stating that a finding the rubric's `Do not flag` list excludes is disputed on the `premise false` ground.
Leave the Auto-mode assumption as the record of that pick, or point it at D2.

### [LOW] D2 does not say whether "every change" includes changes outside the write surface

**Location:** `_lyx/discussion/decision-record.md:45` (D2) against `:61` (D3).

**Issue:** D1 orders Dispute before Fix with own change, and D2 lets the fixer dispute when "every change that addresses it makes the target worse".
D3 says an own change that "would need to leave" the fix-scope write surface is deferred.
Take a finding whose only sound change lies outside the write surface.
Read over in-surface changes only, D2 makes it a dispute.
Read over all changes, D2 does not apply and D3 defers it.
D1 checks Dispute first, so the stricter reading wins by order, and a defect that only needs another write surface is disputed instead of deferred to the operator.

**Fix:** In D2, state that "every change" includes changes outside the fixer's write surface.
A finding whose only sound change leaves the surface is then deferred under D3, never disputed.

### [LOW] D4 does not place the dispute answer within the clean-room sequence

**Location:** `_lyx/discussion/decision-record.md:70` (D4).

**Issue:** `priorRoundsBlock` (`internal/burlerengine/prompt.go`) tells the reviewer to form its own findings first and to read the prior fixer-reports only after its review is saved.
D4 requires a re-raised disputed finding's `**Issue:**` to state why the dispute's evidence does not hold.
That answer can only be written after the reviewer reads the fixer-report, which means editing the saved review.
That edit must also happen before the reviewer ends its turn, because the ready marker that releases the fixer is written then.
D4's rule is appended to that block's closing sentence without saying it amends the saved review.
So the plan writer has to choose between amending the review and filing a new finding, and the stencil wording follows from that choice.

**Fix:** State in D4 that once the reviewer has read a prior fixer-report, it amends the matching finding's `**Issue:**` in its saved review before it ends its turn, and that the clean-room order is otherwise unchanged.

## Focus departures

None.
Verdicts on the focus items that produced no finding:

- D1 against D2 and D3: "every fix worse" and "some other change resolves the premise" exclude each other, and D1's order settles a change that resolves the premise but harms the target as a dispute.
  The one remaining overlap is the write-surface case in F4.
- Scope and D6 against Acceptance criteria: every file D6 edits is in Scope.
  `docs/overview.md` line 496 is the Burler line that states the false-premise-only rule.
  D1 to D4 and D6 each have an acceptance criterion.
  D5 changes nothing, and Scope's Out list carries it, so it needs no criterion.
- Relocation check: the Decisions and Constraints carry Decision and Rationale text only, and no paragraph is deliberation narrative that belongs in `support-log.md`.
