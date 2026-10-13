# Card 3 — bouncer-hard-cap

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#NewBouncer`
- `internal/shedadapters#Bouncer.settleUnconverged`
- `internal/shedadapters#Bouncer.escalationReason`
- `internal/shedadapters#Bouncer.renderEscalation`
- `internal/shedadapters/doc.go`
- `contracts/stencils/bouncer/bouncer-template-escalation.md`
- `contracts/stencils#registry`
- `contracts/stencils#wordingClaims`
- `contracts/specs/refusal-spec.md`
- `internal/shedadapters#TestNewBouncer_ValidationRules`
- `internal/shedadapters#TestBouncer_Escalation_AwaitsWithTheRecordAndLeavesItUntouched`
- `internal/shedadapters#TestBouncer_Escalation_RecordedDecision`

**Create:**
- `internal/shedadapters#Bouncer.atHardCap`
- `internal/shedadapters#Bouncer.retireContinue`
- `contracts/stencils/bouncer/bouncer-fragment-decide-open.md`
- `contracts/stencils/bouncer/bouncer-fragment-decide-capped.md`
- `contracts/stencils#BouncerFragmentDecideOpen`
- `contracts/stencils#BouncerFragmentDecideCapped`

**Intent:** The Bouncer learns its perch's hard cap and never continues a round at or past it (D3), a continue past the soft cap is budget-exempt whoever recorded it (D4), and an escalation at or past the hard cap offers only accept (D6).
The soft-cap rule itself is card 5; this card adds the cap, the recorded-continue handling and the escalation surfaces both cards share.

`BouncerConfig` gains `HardCap int`: the perch's hard cap as a round number, counted from 1 in the segment's current generation, so a round numbered `HardCap` or higher is at or past it.
0 is the absent value and means no hard cap, which keeps every caller that tells none behaving as before.
`NewBouncer` refuses a negative `HardCap` with "shedadapters: NewBouncer: HardCap must not be negative, got %d", a wiring guard reached only by a caller bug since config load refuses a hard cap below 1 (cards 6 and 7).
`NewBouncer`'s budget-rule doc paragraph replaces "only a budget-cause continue is exempt" with the D4 rule below, and adds that no round at or past `HardCap` is ever continued.

`Bouncer.atHardCap(round)` reports `b.cfg.HardCap > 0 && round >= b.cfg.HardCap`, reading the round number alone and never Shed's count, since a goto resets that count while round numbers carry on (D3).

`Bouncer.settleUnconverged`'s recorded-continue branch:

- At or past the hard cap, whatever the decision's cause, it calls `retireContinue(round)` and then `b.escalate(round, EscalationHardCap, ptr)`, so the round returns `Awaiting` with a fresh record, brief and notice of cause `hard-cap` instead of running round N+1 (D2's paragraph on a recorded continue at the hard cap).
  A failed `retireContinue` returns an error, as `escalate`'s own failed write does: "shedadapters: <name> (bouncer): retire the continue decision of round N at the hard cap: <err>; way forward: make the named path writable, then run `lyx loom start`, which re-escalates the round".
- Below the hard cap the branch returns Stuck as today, with `ptr.BudgetExempt` true when the cause is `budget` or when `b.budgetReached(round)` holds (D4).
  A circling-cause continue below the soft cap stays counted.

`Bouncer.retireContinue(round)` moves the round's escalation record, its parent notice and then its decision file aside through `archiveStaleOutputs`, in that order, each to the timestamped sibling `archiveStaleOutputs` names, which no reader opens.
Absent files are skipped, so a round whose continue answered a legacy CIRCLING with no record moves only its decision file, and a crash between the moves leaves a state the next call finishes: the decision file is still there, so the branch runs again.
The doc comment states that a replay after the moves and before the fresh record finds no decision file and no record and is settled as an undecided round (the rule, from card 5).

`Bouncer.escalationReason`: on a round where `atHardCap` holds, whatever the cause, the decide clause names only `lyx loom circling accept<slug>` and says the round is at or past the perch's hard cap so it can only be accepted, then `lyx loom resume`.
Below the hard cap it reads as today.

`Bouncer.renderEscalation`:

- It reads `bouncer-fragment-decide-capped` when `atHardCap(round)` holds and `bouncer-fragment-decide-open` otherwise, fills it with `slug`, and passes the result as the brief's new `decide_options` marker.
  Go picks a stencil and interpolates values only, so the wording stays in stencils (`PATTERN-stencil-ownership`).
- It fills the brief's new `judge_stencil_path` marker with `stencilstore.Path(b.cfg.StencilsDir, "bouncer-template-judge")`.
- A failed fragment read or fill returns an error like the brief's own, which `escalate` already turns into the plain-Reason degrade.

`bouncer-template-escalation.md`:

- The header comment says Go interpolates only paths, round, cause, segment and slug, plus the decide fragment it picks by the hard cap.
- The cause list names three causes.
  `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
  `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
  `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- A sentence points at `{{.judge_stencil_path}}` for what a ledger entry's `design-wrong` and `mechanical` tags mean, restating neither (`PATTERN-producer-pointer-rule`).
- Step 2 becomes `{{.decide_options}}`; steps 1 and 3 and the cannot-settle paragraph stay.

`bouncer-fragment-decide-open.md` holds today's two options: `lyx loom circling accept {{.slug}}` ending the segment unconverged on this round, recorded as unconverged, and `lyx loom circling continue {{.slug}}` running one more round.
`bouncer-fragment-decide-capped.md` holds accept alone, with the sentence that this round is at or past the perch's hard cap, so the segment can only be accepted or handed on.
Each opens with a comment naming its marker `{{.slug}}` and that `renderEscalation` picks it.
`contracts/stencils/stencils.go` embeds both as `BouncerFragmentDecideOpen` and `BouncerFragmentDecideCapped` beside `BouncerTemplateEscalation`, and `registry` lists both after `bouncer-template-escalation`.

`wordingClaims`:

- The `bouncer-template-escalation.md` row keeps `lyx loom decision add`, `lyx loom resume`, the no-`lyx loom start`, no-`darn`, design-call and record-nothing claims.
  Its accept and continue claims move to the fragments.
  "the brief names both causes" becomes three must-claims, `` `circling` ``, `` `budget` `` and `` `hard-cap` ``, plus "could not read its inputs" for D6's new meaning of `budget`, and it gains must-claims `{{.decide_options}}` and `{{.judge_stencil_path}}`.
- A new `bouncer-fragment-decide-open.md` row must-claims `lyx loom circling accept {{.slug}}`, `lyx loom circling continue {{.slug}}` and "recorded as unconverged".
- A new `bouncer-fragment-decide-capped.md` row must-claims `lyx loom circling accept {{.slug}}` and "accepted or handed on", and must-not-claims `lyx loom circling continue`.

`refusal-spec.md`'s "escalation record write failed" row widens its trigger to "or fails to move a continue decision recorded at or past the perch's hard cap aside"; its way forward stays.

`internal/shedadapters/doc.go`'s Escalation paragraph gains the hard cap: `BouncerConfig.HardCap`, the `hard-cap` cause, a recorded continue at or past it moved aside and re-escalated, the accept-only Reason and decide fragment, and the D4 exemption replacing "marked BudgetExempt exactly when the escalation's cause is budget".

Tests:

- `TestNewBouncer_ValidationRules` gains a row "NegativeHardCap" setting `HardCap` to -1 and wanting "HardCap".
- `TestBouncer_Escalation_AwaitsWithTheRecordAndLeavesItUntouched` gains a `hardCap int` field set on the config through its tweak.
  A new row "a spent budget at the hard cap offers only accept" runs CONTINUE, `spentBounces`, hard cap 2, wanting cause `budget`.
  For a row whose round 2 is at or past its hard cap, the Reason check requires `lyx loom circling accept my-task` and forbids `lyx loom circling continue`, and the brief must hold "accepted or handed on" and no `lyx loom circling continue`.
  For every other row, the brief must hold `lyx loom circling continue my-task`.
  The `//testtiming:keep` reason gains "and the accept-only offer at the hard cap".
- `TestBouncer_Escalation_RecordedDecision` gains `hardCap int` and `preRecord EscalationCause` fields.
  Its doc comment and `//testtiming:keep` reason replace "exempt from the budget only for a budget cause" with "exempt for a budget cause or at a spent budget", and add the re-escalation at the hard cap.
  `preRecord` writes round 2's escalation record with that cause through `writeEscalation` before the first Call, which then returns the record's Awaiting.
  New rows:
  - "continue over a circling escalation at a spent budget is exempt": CIRCLING, `spentBounces`, `preRecord` circling, continue, cause circling, `wantExempt` true.
    It fails if the exemption still reads the cause alone.
  - "continue over a budget escalation at the hard cap re-escalates hard-cap": CONTINUE, `spentBounces`, hard cap 2, continue, cause budget.
  - "continue over a circling escalation at the hard cap re-escalates hard-cap": CIRCLING, `belowBounces`, hard cap 2, `preRecord` circling, continue, cause circling.
    The `preRecord` keeps the first Call's cause `circling` once card 5's rule settles an unrecorded round at the hard cap.
  For the two hard-cap rows, a `wantReEscalate` flag makes the Call after the continue require `Awaiting` with "cause: hard-cap" in the Reason and no `lyx loom circling continue`.
  It also requires that `readEscalation` reads cause `hard-cap` with a notice, that no file is at `circlingDecisionPath(runDir, 2)`, and that `filepath.Glob` finds one retired sibling each for `round-2-escalation-*.md`, `round-2-parent-notice-*.md` and `round-2-circling-decision-*.md`.
  Then `RecordCirclingDecision(runDir, CirclingAccept)` returns round 2 with cause `hard-cap`, and the next Call returns Done with the decision read back settled with cause `hard-cap`.

**ImpactSummary:** Bouncer-internal: one optional config field, one changed branch of the recorded-decision path and the escalation's Reason and brief render; callers that tell no hard cap behave as before except a continue past the soft cap is now exempt whatever its cause, and the brief stencil's marker set grows by two fragments' worth.

**Commit:** 3: Bouncer hard cap: a recorded continue there re-escalates, escalations there offer only accept
