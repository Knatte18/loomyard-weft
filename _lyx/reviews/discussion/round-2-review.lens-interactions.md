```yaml
lens: interactions
outcome: reported
```

Task: interactions-lens review of the decision record's decisions, pair by pair.

**Pairs checked**
- D7 × D8 (class definition × escalation on the class): broken. A parent `accept` is not an answer D7 recognises, so the same question escalates again (F2).
- D8 accept-applies-refusal × the acceptance criterion "a BLOCKING `decision` finding alone does not refuse the accept": broken. The ledger half of the refusal is not exempted (F1).
- D8 escalation-before-refusal × D8 `continue` path (standsAsRecorded replay): broken. A `continue` settles over a judgment the refusal rejects (F3).
- D8 accept-below-cap-runs-as-continue × the soft-cap rule's circling check on the previous round's decision: broken, LOW (F4).
- D9 × D7 (writer's Open risks line × answered-question rule): held, but feeds F2 on a later generation.
- D9 × D8 accept carry-over (writer line plus carried finding in `## Open risks`): held. It produces a duplicate line, which is no defect of its own.
- D5/D6 × D7 (lens parent questions × class cases): held.
- D6 × D1 (decision-note reads × surviving `DecisionNoteTitle`): held.
- D6 × D2 (lenses use `lyx loom peers`, gate removed): held.
- D8 hard-cap accept retire plus re-judge × D8 accept-only at the hard cap: held. It halts at budget, as the record states and as a refused CONVERGED does today.
- D8 × arbiterstop default refusal: held. A non-hard-cap, non-circling cause is already refused by the `default` arm.
- D8 settled-accept re-entry × the existing `reenterUnchanged`/`resettle` paths: held. A settle record is live by generation and later rounds, not by its Settled flag.
- D8 fix-row switch × D8 Bouncer switch (one notice per round): held. The fix row must keep `DecisionsRouted` true without a notifier, which the record states.
- D10 × D8 scope-out (darn Code-Review keeps today's notice): held.
- D10 × D1 (decision-note title form survives): held.
- D2 in-flight × D3 (retired keys of a blocked run): held.
- D4 × D11 (reportengine keeps the parent-review parse and the `Merge-In` classification): held.
- D2/D11 × the `lyx loom peers` row-position constraint: held.
- D11 attempt-record fallback × `rowEntry` over the renamed history: held. `RenameProducers` rewrites history, so the entry index matches.

**Findings**

F1. BLOCKING, design, `decision-record.md` D8 second bullet against the acceptance criterion "a BLOCKING `decision` finding alone does not refuse the accept".
D8 leaves the round's `decision` findings out of the refusal's "BLOCKING-finding check" only, which is the review-file check.
`judgmentRefusal` also refuses a CONVERGED verdict over any ledger entry open at severity BLOCKING, whatever its class (`internal/shedadapters/bouncer.go` `judgmentRefusal`, first loop).
A BLOCKING `decision` finding is never fixed, because the fixer is barred from it, so its ledger entry stays open at BLOCKING.
Every accept is therefore refused.
Below the hard cap the accept becomes a continue, the next round re-raises and re-escalates, and at the hard cap the accept is retired into a re-judge that halts at budget.
The parent has no way to pass a BLOCKING `decision` finding.
`ledgerEntry` carries `Class` (`bouncerfiles.go` `ledgerEntry`), so the exemption is expressible, but D8 does not state it.

F2. MEDIUM, design, D8 accept below the hard cap, read together with D7 and D9.
A parent `accept` creates no `Added after Discussion` entry and no decision note, so under D7 it is no answer.
Below the cap the accept runs as a continue with no carry-over written.
The next round's lenses re-raise the same parent-only question, and the Bouncer escalates it again with cause `decision`.
The parent is re-asked every round until the hard cap.
Likewise, after a settled accept the carried `## Open risks` finding, and a D9 writer line, are still unanswered parent questions.
A later amendment or new generation re-raises them as `decision` and stops the run again.
D8 never says how the parent's accept is remembered across rounds or generations.
Evidence: D8 bullets 2 and 6, D7 text, and `acceptRound` (carry-over is written only on settle).

F3. MEDIUM, design, D8 first and second bullets.
The escalation runs before the refusal for every verdict, but only an `accept` re-applies the refusal.
A recorded `continue` reaches the round through `standsAsRecorded`, so `Call` settles it with no refusal (`bouncer.go` `Call` tail and `standsAsRecorded`).
The refused judgment then stands as the base of the next round: a ledger that dropped a previous key, or a CONVERGED with an open BLOCKING entry.
The next round's dropped-key check compares against that defective ledger, so the dropped key is lost for good.
The same holds for an accept below the cap converted to a continue: no re-judge repairs the ledger.
D8's "What it can skip or let through" bounds only the accept, and states no bound for this bypass of the dropped-key guard.

F4. LOW, consistency, D8 "Below the hard cap the accept runs as a `continue`".
D8 does not say whether the round's decision file stays an `accept` or is rewritten.
`softCapRule` detects circling only when the previous round's decision is `CirclingContinue` (`bouncer.go` `softCapRule`, `previousDecision == CirclingContinue`).
If the file stays an `accept`, a design-wrong key open across that pair of rounds escapes the circling escalation.

Tool calls: none; everything I needed was already in context.
