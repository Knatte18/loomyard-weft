# Fixer report — discussion, round 2

Every finding's premise was checked against `internal/shedadapters/bouncer.go` (`judgmentRefusal`, `settleUnlessRefused`, `standsAsRecorded`, `settleUnconverged`, `retireContinue`), `bouncerfiles.go` (`ledgerEntry.Class`), `burlerfix.go` (`BurlerFixProducer.Call`), `contracts/recipes/darn-recipe.yaml` and `internal/darnengine/doc.go`; each holds.

## Changes

- F1 (BLOCKING), fixed as proposed: D8's second bullet leaves the round's `decision` findings out of both BLOCKING checks, the review's and, by the entry's class, the ledger's; criterion 220 adds a BLOCKING `decision` finding whose ledger entry the CONVERGED round holds open.
- F2 (MEDIUM), fixed as proposed: a decision recorded on a `decision` escalation whose judgment the refusal rejects takes the re-judge first, at the harvest and on a replay, archiving the judge outputs, spending one counted bounce unit, and keeping the escalation record and decision file; the What-it-can-skip line covers the continue; a new criterion covers a continue over a ledger that drops a previous key.
- F3 (MEDIUM), fixed differently: the proposal carried a refused accept's findings into the next round's focus file. I made a different change and removed the path the finding attacks: an accept is never turned into a continue, it stays on disk across the re-judge and acts once a judgment is admitted, so the parent's answer is never lost and the round is not re-escalated. Carrying findings into a focus file would have added a second carrier for an answer the decision file already holds. Criterion 220 states that the round is not re-escalated.
- F4 (MEDIUM), fixed differently: the proposal named a carrier for "whose fixer gets the BLOCKING findings the refusal named". That sentence is gone along with the accept-as-continue path (F3). A BLOCKING finding reaches a fixer only through a `continue`, which the brief now offers for that purpose.
- F5 (MEDIUM), fixed as proposed in substance: the snapshot of `Added after Discussion` titles is a round file of its own, written only when absent and moved neither by `retireContinue` nor by a re-judge, so it also survives a crash between the archive and the fresh record. The retired-accept-at-cap case is gone with F3's change. Criterion 222 adds the hard-cap snapshot.
- F6 (MEDIUM), fixed as proposed: D10's no-parent bullet decides the gate behaviour from `darn-recipe.yaml`'s `discussion` gate (`attempts: 3`) and the header's exhausted-gate halt: a re-prompted writer ends its turn unchanged, so the gate exhausts and halts the run. The third Notes bullet is deleted, and criterion 225 names the unchanged turn end.
- F7 (LOW), fixed as proposed: D10's bound now states `writer_timeout_min` as the writer's whole-session deadline, that a timeout halts the run with no `on_stuck`, and that a resume, not an automatic respawn, repeats the check (refined in the self-check).
- F8 (LOW), fixed differently: the proposal rewrote a refused accept on disk as a continue with cause decision. With F3's change no accept turns into a continue, so the soft-cap rule reads a real `accept` or `continue` decision file and nothing escapes its circling check.
- Support log: two rejected alternatives added (accept-as-continue, and a new gate refusal for the no-parent stop).

## Self-check

- fix-consistency, LOW, the Notes name only `settledGeneration`, not the `standsAsRecorded` replay rule D8 now changes: fixed as proposed (the Notes' second bullet names both).
- fix-way-forward + fix-proposal, MEDIUM (merged), a refusal that holds over a kept decision has no way forward, since a decision file cannot be replaced (`RecordCirclingDecision`, `ErrCirclingDecided`), and an accept could pass a BLOCKING `design` finding without telling the parent: fixed with my own change. A `continue` now applies only the dropped-key check. A refusal that still holds once the budget is reached retires the decision and re-escalates with cause `decision`, its brief naming the held refusal. The brief states that an accept passes every BLOCKING finding it lists and that a continue sends them to a fixer. Criterion 220 and the bound line are updated to match.
- fix-proposal, MEDIUM, a ledger entry's class is optional, so an unclassed BLOCKING `decision` entry still refuses: fixed with my own change. D8 states that an unclassed open BLOCKING entry still refuses, which the judge stencil already rules out (`bouncer-template-judge.md`: every open entry carries its class and severity). Matching entries to findings by key was not done, because ledger keys are not review finding ids.
- fix-claims, LOW, a resume can re-attach to the waiting writer rather than repeat the check: fixed as proposed (D10's bound names both resume cases).
- fix-way-forward + fix-proposal, LOW (merged), the no-parent halt does not tell whoever resumes it how to proceed: fixed as proposed (the stop report names the way forward: a `Decision for <slug>:` note, then `lyx loom resume`).

## Disputed

None.

## Deferred

None.
