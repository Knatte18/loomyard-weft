---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (harvest and replay paragraph), D8, Acceptance criteria (replay and non-Merge auto-accept bullets)
    summary: On an auto-accept perch a harvest of a round with a design-wrong entry open accepts without the rule (D8), while a replay of the same unsettled round goes to the rule (D8 first bullet) and continues or escalates, so the acceptance criterion that a replay settles as the harvest would, auto-accept perches included, contradicts D8, and D2's claim that a harvest and a replay settle the same way is false on those perches.
    fix: Pick one outcome for an unsettled replay on an auto-accept perch and state it in D8 and the Acceptance criteria. Either make the replay take the same auto-accept the harvest takes (drop D8's first bullet and keep the criterion), or keep the bullet and rewrite the criterion to say a replay of an unsettled round on an auto-accept perch is settled by the rule, not as the harvest would; in the second case also narrow D2's sentence to say harvest and replay agree on perches without auto-accept.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (soft cap and steps 3-5), D3 (last paragraph), D4
    summary: The soft cap is reached by Shed's counted Stucks, which also count the seed, every degrade and every rejudge, while D3 says the hard cap counts judged rounds; the record never says what at the soft cap, past the soft cap and at the hard cap are measured by, so the budget can be reached at a round below the soft cap, D4's exemption may then not apply and Shed blocks the rule's continue, and a hard cap equal to the soft cap does not mean no round past the soft cap.
    fix: In D2 and D3 define each cap's measure in one place, for example that the rule's window opens when the Bounces seam reports the budget reached, that the hard cap compares the round number within the segment's generation, and that past the soft cap means a round after the one at which the budget was first reached; make D4's exemption condition use the same measure as the rule's window, and correct D3's sentence about equal caps to match.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 step 4, Auto-mode assumptions A4
    summary: Step 4's test of a design-wrong key also open in the previous round's ledger does not say that keys match by the ledger key, nor whether the previous round's entry must itself be design-wrong (tagged design-wrong and BLOCKING or MEDIUM) or only open, so a key that was LOW, NIT, tagged mechanical or untagged in the previous round has no defined outcome; it also looks only at the previous round, where the existing earned-CIRCLING test looks at any earlier round, without saying the narrowing is intended.
    fix: Rewrite D2 step 4's second condition to state that two entries are the same when their ledger keys are equal, and whether the previous round's entry must be open only or open and counting as design-wrong by D2's own definition (with a missing tag reading as design-wrong per D1); state that only the immediately previous round is consulted, and add an acceptance criterion for a key that was LOW or tagged mechanical in the previous round.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1, D2 Bound, Open risks first bullet
    summary: The rule accepts unreviewed any BLOCKING or MEDIUM fix the judge tags mechanical, but D2's Bound only restates the tag condition and never says what a mis-tag lets through or what bounds it; the only mention is the Open risks line, which the rubric's attack-surface test does not accept in place of the Decision's own bound.
    fix: Extend D2's Bound to state that a finding mis-tagged mechanical lets an unreviewed design change through, and name what bounds it, for example that this is reachable only at or past the soft cap, that every accepted open entry is carried over into the decision record's Open risks with its tag, and which later review segment re-reads the artifact; if nothing bounds it beyond visibility, say so in the Bound.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2 step 1, D5, D6
    summary: Step 1 runs before step 3, so a round at or past the hard cap whose inputs are unreadable escalates with cause budget, and D6 restricts the Awaiting Reason to accept only at cause hard-cap, so that budget escalation offers lyx loom circling continue, which D5 refuses there.
    fix: In D6 make the restriction follow the round, not the cause - at a round at or past its perch's hard cap the Awaiting Reason and the brief offer only accept, whatever the cause - and add a matching acceptance criterion for a budget escalation at the hard cap.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (decision-file paragraph), Acceptance criteria (recorded continue at the hard cap)
    summary: A round whose recorded continue sits at or past the hard cap is escalated with cause hard-cap, but that round already has a decision file, so lyx loom circling accept is refused as already decided and the pending-escalation check reads it as answered; the escalation's only offered way forward cannot run.
    fix: In D2 state how that escalation is answered, for example that the Bouncer retires the stale continue decision file before writing the hard-cap escalation record so circling accept can record a decision, or that it accepts the round directly; name the way forward in the decision and add it to the acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md Scope (orch role stencil item)
    summary: Scope puts the orch role stencil's circling continue guidance and its small-task review_max_bounces 2 advice in scope, updated for the hard cap and the rule, but no Decision says what the new guidance is, so Plan-Write, which reads only the decision record, would have to make that design call itself.
    fix: Add a Decision, or a sentence in D5 or D6, stating the orch stencil's new wording in substance - when the orch may continue (below the hard cap only, accept otherwise) and whether the small-task advice keeps review_max_bounces 2 and also sets a hard cap - with its rationale.
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D2 step 5, D6
    summary: Step 5 records an automatic continue decision with no escalation behind it, but D6 says a decision file copies its escalation's cause, so the cause the rule's continue carries is unstated.
    fix: In D2 step 5 name the cause the automatic continue decision records (for example budget, as the automatic accept already does), and note in D6 that rule-written decision files carry that cause rather than an escalation's.
---

### [BLOCKING] Replay of an unsettled round on an auto-accept perch has two outcomes

**Location:** `_lyx/discussion/decision-record.md` D2, D8, Acceptance criteria
**Issue:** D8 keeps auto-accept on a harvest "without the rule" and sends a replay of a round with no decision file and no escalation record to the rule.
On an auto-accept perch with an open BLOCKING or MEDIUM `design-wrong` entry, the harvest accepts (the criterion "A non-Merge auto-accept perch still accepts a harvested round at the soft cap with a design-wrong entry open" pins it) while the replay continues or escalates.
The criterion "a replay of an unsettled round settles as the harvest would, auto-accept perches included" therefore cannot hold together with D8, and D2's "a harvest and a replay of the same round settle the same way" is false on those perches.
Today's code (`settleUndecided` in `internal/shedadapters/bouncer.go`) gates the auto-accept on `spawned`, which is the source of the asymmetry the record keeps.

### [MEDIUM] Soft and hard caps are measured by different, undefined counters

**Location:** `_lyx/discussion/decision-record.md` D2, D3, D4
**Issue:** `budgetReached` compares Shed's episode Stuck count with the budget (`Routing.Bounces` in `internal/shedengine/progress.go`), and the `NewBouncer` budget rule counts the seed Stuck, every degrade and every rejudge.
D3 says the hard cap counts judged rounds.
After one rejudge the budget is reached a round early; the rule then runs, and D4's "at or past the soft cap" exemption is ambiguous there, so the rule's continue may be counted and Shed blocks the run.
The hard cap cannot be measured by Shed's count either, since exempt continues never raise it.
"Past the soft cap" in step 4 and D3's "a hard cap equal to the soft cap … means no round past the soft cap" both depend on this undefined measure.

### [MEDIUM] Step 4's returning-key test is underdefined

**Location:** `_lyx/discussion/decision-record.md` D2 step 4, A4
**Issue:** "A design-wrong key was also open in the previous round's ledger" can be read as the key being design-wrong now and merely open before, or design-wrong in both rounds.
Previous-round entries at LOW or NIT, tagged `mechanical`, or untagged have no stated outcome.
The existing earned-CIRCLING evidence (`circlingEvidence` in `bouncerfacts.go`) looks at any earlier round, not only the previous one; the record does not say whether the narrower window is deliberate.

### [MEDIUM] The mis-tag skip has no bound in the Decision

**Location:** `_lyx/discussion/decision-record.md` D1, D2 Bound
**Issue:** The rubric's attack-surface test asks the introducing Decision to state what a new escape hatch lets through and what bounds it.
D2's Bound restates the tag condition, which is the hatch itself; what a wrong `mechanical` tag lets through, and what limits it, appears only in Open risks.

### [MEDIUM] A `budget` escalation at the hard cap offers a refused continue

**Location:** `_lyx/discussion/decision-record.md` D2 step 1, D5, D6
**Issue:** The ordering puts the unreadable-input check first, so a hard-cap round can escalate with cause `budget`.
D6 limits the Awaiting Reason to accept only at cause `hard-cap`, and today's `escalationReason` offers both verbs; the `budget` Reason at the hard cap therefore names `circling continue`, which D5 refuses.
D7's `budget` rule ("as for `circling`") carries the "below the hard cap" condition, so only the Reason and the brief disagree.

### [MEDIUM] A hard-cap escalation over a recorded continue cannot be answered

**Location:** `_lyx/discussion/decision-record.md` D2
**Issue:** `RecordCirclingDecision` creates the decision file with `O_EXCL` and returns `ErrCirclingDecided` when one exists, and `PendingEscalation` reports a round with a decision file as not pending.
A recorded `continue` at or past the hard cap is a round that already has a decision file, so the new `hard-cap` escalation's only offered verb, `circling accept`, is refused and the run has no way forward.
Such a round arises when the hard cap is lowered in config after a continue is recorded, or from a decision an older binary wrote.

### [MEDIUM] The orch stencil change has no decision

**Location:** `_lyx/discussion/decision-record.md` Scope
**Issue:** `contracts/stencils/orch/orch-template-role.md` advises a small task's `review_max_bounces: 2` and `circling continue` to grant more rounds.
Scope says both are updated for the hard cap and the rule, but no Decision states the new guidance, so Plan-Write would design it.

### [LOW] The rule's automatic continue has no stated cause

**Location:** `_lyx/discussion/decision-record.md` D2 step 5, D6
**Issue:** `readCirclingFile` requires a decision file's cause to be one of the legal causes, and D6 says the decision file copies its escalation's cause.
The rule's continue has no escalation, so its recorded cause is left to the plan writer.

## Focus verdicts

- D2 against D8 and the replay criterion: not consistent (F1). Of the round states, a decision file and an escalation record each map to one outcome, apart from the recorded-continue case at the hard cap (F6). Neither maps to the rule, except a harvest on an auto-accept perch.
- Step 4's key test: underdefined (F3).
- The mis-tag bound: missing from the Decision (F4).
- D4 and D5: the hard cap, the rule and the `circling continue` refusal bound the rule's continue, a parent's or arbiter's continue below the hard cap, and equal caps (step 3 runs before step 4, so the soft-cap round escalates `hard-cap`). That holds once the cap measures are defined (F2).
- D3: the default hard cap of 4 sits above every default soft cap (loom 3, Merge 1, darn 1). The refusal and the parked-run open risk agree. Darn's soft cap of 0 runs no review round, so it leaves no path past the hard cap.
- D6 and D7: `hard-cap` and `circling` are consistent across the Reason, the brief and the arbiter rule. `budget` is not consistent at the hard cap (F5). D7's `budget` rule is conditioned on `arbiter-when` and does not contradict Scope's unchanged eligibility list.
- Scope, Decisions and Acceptance criteria: the orch stencil item exists only in Scope (F7). Nothing in `decision-record.md` is deliberation narrative that belongs in `support-log.md`. No requirement exists only in `support-log.md`.
