# Card 12 — bouncer-decision-escalation

**Create:**
- `internal/shedadapters#EscalationDecision`
- `internal/shedadapters#Bouncer.routeDecisions`
- `internal/shedadapters#Bouncer.decisionFindings`
- `internal/shedadapters#Bouncer.decisionRefusal`
- `internal/shedadapters#Bouncer.escalateWith`
- `internal/shedadapters#Bouncer.renderDecisionDetail`
- `internal/shedadapters#DecisionHaltPrefix`
- `internal/shedadapters#decisionSnapshotPath`
- `internal/shedadapters#writeDecisionSnapshot`
- `internal/shedadapters#addedSinceSnapshot`
- `contracts/stencils#BouncerFragmentDecision`
- `contracts/stencils/bouncer/bouncer-fragment-decision.md`
- `internal/shedadapters#decisionBouncer`
- `internal/shedadapters#TestBouncer_DecisionEscalation`

**Edit:**
- `internal/shedadapters#EscalationCause.valid`
- `internal/shedadapters#ErrEscalationMalformed`
- `internal/shedadapters#writeEscalation`
- `internal/shedadapters#readEscalation`
- `internal/shedadapters#readCirclingFile`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.settleUnlessRefused`
- `internal/shedadapters#Bouncer.standsAsRecorded`
- `internal/shedadapters#Bouncer.settledGeneration`
- `internal/shedadapters#Bouncer.escalate`
- `internal/shedadapters#Bouncer.renderEscalation` -> `func (b *Bouncer) renderEscalation(round int, cause EscalationCause, detail string) (brief, notice string, err error)`
- `internal/shedadapters#Bouncer.escalationReason`
- `internal/shedadapters#Bouncer.buildCarryOver`
- `internal/shedadapters#TestRecordCirclingDecision_WritesPendingFile`
- `internal/shedadapters/doc.go`
- `contracts/stencils/bouncer/bouncer-template-escalation.md`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Extends:**
- `contracts/stencils#entries`

**Uses:**
- `internal/shedadapters#newBouncerFixture`
- `internal/shedadapters#withNestedRunDir`
- `internal/shedadapters#withShuttle`
- `internal/shedadapters#withEscalationInputs`
- `internal/shedadapters#layoutBouncerRun`
- `internal/shedadapters#bouncerVerdictContent`
- `internal/shedadapters#taggedLedgerContent`
- `internal/shedadapters#circlingSeams`
- `internal/shedadapters#carryOverRecorder`
- `internal/shedadapters#judgeFakeShuttle`
- `internal/shedadapters#newReentry`
- `internal/shedadapters#convergingShuttle`
- `internal/shedadapters#reentryOption`

**Intent:** D8: a Bouncer told `BouncerConfig.EscalateDecisions` escalates a judged round whose accepted review holds a `decision`-class finding to the run's parent, with the new cause `decision`, whatever the judge ruled.
Card 13 sets the switch on loom's Discussion-Bouncer only; with it false every path below is skipped and the Bouncer behaves exactly as before.

**Cause.**
`EscalationDecision` joins the legal causes: `valid` accepts it, and the three refusals that list the legal spellings (`writeEscalation`, `readEscalation`, `readCirclingFile`) and `ErrEscalationMalformed`'s doc name four.
`RecordCirclingDecision` then records a decision with cause `decision`; `TestRecordCirclingDecision_WritesPendingFile` gains that row.

**Routing.** `routeDecisions` runs first in `settleUnlessRefused`, ahead of `judgmentRefusal`, and returns handled false when the switch is off, the round's review (`ReportName(round)`, parsed by `burlerengine.ParseReview`) does not parse or holds no finding of class `burlerengine.ClassDecision`, the round carries an escalation record of another cause, the round's decision file carries a cause other than `decision` (an automatic `budget` accept or continue, or a decision an older binary recorded), or its decision is a settled accept; each of those takes today's path unchanged.
`standsAsRecorded` returns false for a round whose escalation cause is `decision` while its decision file is absent or not settled, so a replay reaches `settleUnlessRefused` too, and every harvest site (judge harvest, entry-time attach, `entryHarvest`) already calls it; so the routing runs before the judgment refusal, the soft-cap rule, the circling escalation and auto-accept, and neither a re-judge nor `discussion_auto_accept` passes it; a settled `decision` accept stands as recorded on a replay, as today.
`settledGeneration` returns false for a round whose escalation cause is `decision` while its decision file is absent or not settled, even over a CONVERGED verdict, so a re-call returns the same Awaiting and a recorded decision takes the recorded-decision path, never the re-settle, harvest, clear or new generation, whatever `lyx loom decision add` committed meanwhile; a settled accept is a settled generation again, as a settled circling accept is.
Then, by the round's decision file:

- **None.** Write the snapshot when absent, then `escalateWith(round, EscalationDecision, detail, ptr)` and return its Awaiting.
  The detail names the refusal `decisionRefusal(…, CirclingAccept)` returns, when there is one.
- **A pending `continue`.** `decisionRefusal(…, CirclingContinue)` applies only the dropped-key check, since a continue passes nothing.
  A refusal goes to `rejudge` (a counted bounce unit, judge outputs archived, the escalation record and decision file kept), and a dropped-key refusal that holds at the budget then halts the segment through Shed's budget block, as a refused judgment does today.
  With no refusal: at or past the hard cap, `retireContinue` then `escalateWith(…, EscalationDecision, …)`, so the round keeps cause `decision` and its first snapshot; the verb refuses `continue` at the hard cap, so this is reached when a continue recorded below the cap meets a lowered `HardCap` or an amendment generation's smaller cap at settle time; below it, `ensureFocus(round + 1)` and Stuck, `BudgetExempt` when `budgetReached(round)`, which runs another round in the same generation whose fresh reviewers re-review the artifact.
- **A pending `accept`.** `decisionRefusal(…, CirclingAccept)` applies `judgmentRefusal`'s whole check with the round's `decision` findings left out of both BLOCKING checks: a review finding of class `decision` and an open ledger entry of class `decision` do not refuse; an open BLOCKING entry with no class still refuses.
  With no refusal, `acceptRound` passes the round, the carry-over as below.
  With a refusal and the budget not reached, `rejudge`, and the re-judged round reaches the recorded accept again.
  With a refusal at the budget below the hard cap, `retireContinue` retires the accept as it retires a continue, and `escalateWith` re-escalates with cause `decision`, its detail rendered with `held` true: it names the held refusal, says an `accept` re-escalates the round again while the refusal holds, since no budget is left to re-judge it, and names `continue` as the way forward, so the parent can record another answer.
  Over a dropped-key refusal it also says that a `continue` meets the same check, halts the segment at its budget, and that `lyx loom resume` then re-judges the round.
  With a refusal at the budget at or past the hard cap, archive the judge outputs and return Stuck with `OutputPointer.Halt` and a Reason opening with `DecisionHaltPrefix` and naming the segment, the round, the refusal's cause and the way forward: `lyx loom resume`, whose next Call re-judges the round under the recorded accept.
  The halt is budget-exempt, since a budget halt would persist the shed-wide budget reason and drop this one; card 14 keeps a halt so prefixed with the run's parent.
- **A settled accept** takes today's path through the handled-false return above.

**Snapshot.** `writeDecisionSnapshot` writes `round-<N>-decision-snapshot.yaml` holding the titles of the decision record's `Added after Discussion` entries (`discussionparser.ParseAddedDecisions` over `DecisionRecordPath`; an empty or absent record writes an empty list), only when absent; `retireContinue` and `rejudge` move it not, so a re-escalation keeps the first snapshot.
`addedSinceSnapshot` returns the titles the record holds now that the snapshot lacks, in record order.

**Carry-over.** `buildCarryOver`, for a round whose escalation cause is `decision` and whose decision is an accept, adds every `decision` finding of the round's review at any severity, keyed by the finding's id, with the review's severity, rounds `[round]` and `AddedSince` from `addedSinceSnapshot`, whatever the ledger says, beside the ledger's carried entries; a ledger entry of the same key is not repeated.
It also adds every BLOCKING finding of the round's review, the brief's `blocking_findings`, not already carried by key, each keyed by the finding's id, with its class, severity BLOCKING and rounds `[round]`, whatever the ledger says, so a BLOCKING finding whose ledger entry the judge closed still reaches `## Open risks`.
Both additions match the ledger by exact key only, since a ledger entry records no review id: a finding the judge keyed apart from its review id is listed twice, under each identity, which repeats a risk but drops none.

**Brief.** `escalate` becomes `escalateWith(round, cause, "", ptr)`; `escalateWith` is today's `escalate` with `detail` passed to `renderEscalation`.
`renderEscalation` fills the brief's new `cause_detail` marker, optional through `stencil.FillOptional`, with `detail`.
`renderDecisionDetail` fills `bouncer-fragment-decision` with `slug`, `decision_findings` (one line per finding: id, severity, summary, fix), `blocking_findings` (every BLOCKING finding of the review, or `none`), `refusal` (the cause, or `none`, followed when `held` by the sentence that an `accept` re-escalates the round while the refusal holds and `continue` is the way forward, and over a dropped-key refusal by the sentence that a `continue` meets the same check, halts the segment at its budget, and `lyx loom resume` then re-judges the round) and `decide_verbs`, which Go picks by the hard cap: below it the two lines naming `lyx loom circling continue {{.slug}}` for a round over the amended record and `lyx loom circling accept {{.slug}}` to proceed, at or past it the accept line alone, since the verb refuses `continue` there.
The fragment states: each listed question is one only the run's parent can settle, so put it to the operator unless a recorded operator decision answers it; when a refusal is named, the round is re-judged before the recorded answer acts while budget is left; an `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget; a `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again; the way forward is to record the answer with `lyx loom decision add {{.slug}}`, then the `decide_verbs` line, then `lyx loom resume`.
`bouncer-template-escalation.md` gains the `decision` cause bullet, the `{{.cause_detail}}` line after the cause list and a header naming the new marker; register the fragment in `entries` with its embed var, and `TestStencils_TopLevelMarkers` lists `cause_detail` for the brief and a row for the fragment.
`escalationReason` for cause `decision` opens its decide clause with recording the answer through `lyx loom decision add <slug>`; below the hard cap it offers accept and continue, at or past it accept only, as for every cause.

**Tests.** `TestBouncer_DecisionEscalation`, over `decisionBouncer` (a two-round run directory laid out with `layoutBouncerRun`, round 2's review and ledger as given, with `withEscalationInputs` and `EscalateDecisions` set), one row per criterion of the record:

1. CONVERGED, CONTINUE and CIRCLING verdicts over a review holding a `decision` finding each return Awaiting with cause `decision`, a brief listing the finding's id and the way forward, an Awaiting Reason naming `lyx loom decision add <slug>`, both circling verbs and `lyx loom resume`, no Approve or Commit, and no judge outputs archived; a variant with no decision record on disk writes an empty snapshot and still escalates.
2. A BLOCKING `decision` finding with its ledger entry open over a CONVERGED verdict escalates without a re-judge.
3. `AutoAccept` at a spent budget escalates `decision` instead of accepting: round 2 starts unjudged and a `judgeFakeShuttle` writes its CONTINUE verdict and ledger during the judge spawn, so the call is the harvest auto-accept fires on.
4. A re-call returns the same Awaiting, with and without a changed `Subject` between the calls, over the `newReentry` harness with `EscalateDecisions` set through a `reentryOption`.
5. A recorded `continue` over the CONVERGED round below the budget returns Stuck, not `BudgetExempt`, with round 3's focus file present and the run directory not archived; at a spent budget the same Stuck is `BudgetExempt`.
6. A recorded `accept` returns Done, and the carry-over holds every `decision` finding, a LOW one included, each naming the `Added after Discussion` titles recorded after the escalation; a `decision` finding whose id is also an open MEDIUM ledger key appears once.
7. A CONVERGED round whose review also holds a BLOCKING `design` finding: the brief lists it and the refusal; a recorded accept below the budget is re-judged (verdict, ledger and focus archived, Stuck not `BudgetExempt`) with the escalation record and decision file kept; an admitted accept over a re-judged CONTINUE whose ledger closes that BLOCKING `design` finding returns Done with the finding still in the carry-over.
8. The same refusal at the budget below the hard cap retires the accept and re-escalates `decision` with the refusal, the sentence that an accept re-escalates while it holds, and `continue` in the brief, and the first snapshot kept; a `continue` then recorded returns Stuck with an empty round-3 focus file.
   A held dropped-key refusal at the budget re-escalates with the sentence that a `continue` meets the same check, halts the segment at its budget, and `lyx loom resume` then re-judges the round.
9. A BLOCKING `decision` finding alone, its ledger entry open, does not refuse the accept.
10. A recorded accept at the hard cap over a held refusal at the budget returns a `Halt` Stuck whose Reason opens with `DecisionHaltPrefix` and names the refusal and `lyx loom resume`, writes no new escalation record and archives the judge outputs; the next Call spawns a judge for the round.
11. A recorded `continue` whose ledger drops a key of round 1's ledger is re-judged.
12. An escalation at the hard cap offers accept only, in its Reason and in the brief's `decide_verbs`; a `continue` recorded below the cap, settled after `HardCap` is lowered to the round, re-escalates keeping cause `decision` and the first snapshot.
13. After a settled `decision` accept, a re-entry over an unchanged subject re-settles without a spawn, and over a changed one starts a new generation, over the `newReentry` harness.
14. With `EscalateDecisions` false, a CONVERGED round holding a `decision` finding settles Done as before.
15. With the switch on, a review that does not parse takes today's path: a CONVERGED round is refused and re-judged as before, with no escalation record.
16. A round already carrying a `circling` escalation record keeps it: the Awaiting names cause `circling`.
17. A round whose decision file is an automatic `budget` continue settles as today's continue, with no `decision` escalation written.
18. A replay over a settled `decision` accept whose CONVERGED review holds a BLOCKING `decision` finding returns Done with no re-judge.

What it can skip or let through: an `accept` passes the round over its `decision` findings and any BLOCKING finding listed, but only once a judgment the refusal check admits, and carries every one of them into `## Open risks` as a circling accept does; a `continue` starts the next round only over a ledger that keeps every key of the previous round's.

What bounds it: the `decision` class definition card 8 writes into the rubric is reviewer discipline only, so a finding a reviewer mis-classes `decision` stops the run until the parent answers; one escalation per judged round, plus one per decision retired, either an accept held at the budget below the hard cap or a continue that meets the hard cap at settle time; the hard cap, at which a held refusal halts the run; and the bounce budget each re-judge spends.

**ImpactSummary:** One new cause and one routing step inside the Bouncer, a re-signed private renderer whose one caller moves with it, and one new brief fragment; Plan-Review and Code-Review Bouncers are untouched because the switch is off there.
