---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:112
    summary: D8 exempts decision findings only from the refusal's review check, so the ledger's open-BLOCKING check still refuses every accept over a BLOCKING decision finding, against the acceptance criterion at line 220
    fix: In D8's second bullet, state that the accept's judgment refusal leaves the round's decision-class findings out of both BLOCKING checks, the review's BLOCKING-finding check and the ledger's open-BLOCKING-entry check (ledger entries carry a class), and extend the acceptance criterion at line 220 to a CONVERGED round whose ledger holds the BLOCKING decision entry open.
    origin: lens:interactions
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:112
    summary: A continue on a decision escalation, and an accept run as a continue below the hard cap, start the next round over a judgment the refusal rejects, such as a ledger that drops a key, so the dropped-key guard is bypassed with no stated bound
    fix: In D8, state that a continue or an accept recorded on a decision escalation whose judgment the refusal rejects takes the refusal's re-judge of the round first (archiving the judge outputs, spending one counted bounce unit) and keeps the escalation and decision files for the re-judged round; add the continue case to the What it can skip or let through line, and add an acceptance criterion for a continue over a round whose ledger drops a previous key.
    origin: lens:bound-breaker
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:114
    summary: An accept run as a continue below the hard cap records nothing of the parent's answer, so the next round re-raises the same decision finding and re-escalates every round, and the accepted findings are never carried
    fix: In D8, state how the parent's accept of a round's decision findings survives an accept the refusal turns into a continue, for example by carrying those findings into the next round's focus file as answered by the parent, so the rubric's no-decision rule for answered questions covers them and the next round does not re-escalate them, and by carrying them into Open risks when a later round settles; add an acceptance criterion that a refused accept below the cap does not re-escalate the same finding in the next round.
    origin: lens:interactions
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:114
    summary: The stated mechanism whose fixer gets the BLOCKING findings the refusal named does not exist, since the fix row fixes only its own round's accepted review and round n's fixer ran before the judge
    fix: In D8's second bullet, replace whose fixer gets the BLOCKING findings the refusal named with the actual carrier, for example the refusal's named findings written into round n+1's focus file so the next reviewer re-checks them, or state that the next round reviews afresh and that the refused finding is re-raised only if the reviewer finds it again.
    origin: lens:bound-breaker
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: Both re-escalation paths retire the escalation record through retireContinue into an archive no reader opens, so the snapshot of Added after Discussion titles is lost and the next escalation snapshots titles that include the parent's answer
    fix: In D8's carry-over bullet, state where the snapshot survives retireContinue on the hard-cap continue and on the retired accept, for example read from the record before the archive and written into the fresh escalation record, and add an acceptance criterion that a hard-cap re-escalation keeps the first record's snapshot.
    origin: lens:existing-paths
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:236
    summary: D10's no-parent stop defers to Plan what Darn-Write's gates do with a change description and no record, a fact readable now, the discussion gate re-prompting the writer three times and then halting for a parent that does not exist
    fix: Move the gate behaviour into D10 with its evidence (contracts/recipes/darn-recipe.yaml, Darn-Write discussion gate attempts 3, and the header's exhausted-gate halt) and decide it, for example the stencil tells a re-prompted writer whose stop still holds to end its turn unchanged so the gate exhausts and the run halts blocked with the stop report as its pointer, or a new gate refusal halts at once; then delete the third Notes bullet.
    origin: lens:walkthrough
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:154
    summary: D10's bound claims a respawn after writer_timeout_min repeats the check, but Darn-Write has no on_stuck, so a timeout halts the run, and writer_timeout_min is the whole change's deadline, not a wait timeout
    fix: Rewrite D10's What bounds it line to say that writer_timeout_min is the writer's whole-session deadline, shared with the change, that a timeout halts the run, and that a resume, not an automatic respawn, repeats the check and finds an answer left as a decision note.
    origin: lens:bound-breaker
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:114
    summary: D8 does not say whether an accept run as a continue stays an accept on disk, and softCapRule detects circling only over a previous round whose decision is continue
    fix: In D8's second bullet, state that an accept the refusal turns into a continue is rewritten as a continue decision with cause decision, so the soft-cap rule's previous-round circling check reads it.
    origin: lens:interactions
---

### [BLOCKING] The accept's refusal still fires on the ledger entry of a BLOCKING decision finding

**Location:** `_lyx/discussion/decision-record.md:112`, against the acceptance criterion at line 220.

**Issue:** `judgmentRefusal` (`internal/shedadapters/bouncer.go`) has two BLOCKING checks over a CONVERGED verdict: a ledger entry open at severity BLOCKING, whatever its class, and a review finding at BLOCKING.
D8 leaves the round's `decision` findings out of "its BLOCKING-finding check" only, which reads as the review check.
The fix row withholds a `decision` finding from the fixer, so its ledger entry stays open, and the ledger check refuses every accept over it.
Below the hard cap the accept becomes a continue that re-escalates, and at the cap a re-judge halts at budget, so the parent has no way to pass a BLOCKING `decision` finding.
`ledgerEntry` carries `Class` (`internal/shedadapters/bouncerfiles.go`), so the exemption can be expressed.
Severity is BLOCKING because the hole stops every run whose review holds a BLOCKING `decision` finding, and it contradicts the record's own criterion.
This settles focus item 1: the accept still goes through the refusal, but the stated exemption does not cover the ledger half.

### [MEDIUM] A continue over a refused judgment bypasses the dropped-key guard

**Location:** `_lyx/discussion/decision-record.md:112`, and the bound at line 128.

**Issue:** D8 escalates before the refusal on every verdict, but only an accept re-applies the refusal.
A recorded `continue` reaches the round through `standsAsRecorded`, and `Call` settles it with no refusal (`bouncer.go`, `Call`'s tail).
Counterexample: round 3's judge drops ledger key K, and the review holds a `decision` finding.
The parent records `continue`, round 4's judge carries forward from round 3's ledger, and K is lost for good.
Today `settleUnlessRefused` re-judges that round instead.
An accept below the cap that the refusal turns into a continue has the same hole, since nothing re-judges the round.
D8's "What it can skip or let through" bounds only the accept, so this is an unbounded guard bypass under the rubric's attack-surface question.

### [MEDIUM] A refused accept below the cap forgets the parent's answer

**Location:** `_lyx/discussion/decision-record.md:114`.

**Issue:** `acceptRound` alone writes the accepted carry-over, and an accept the refusal turns into a continue never reaches it.
The parent's accept leaves no `Added after Discussion` entry and no decision note, so under D7 the question is still open.
The next round's lenses raise the same parent-only question, it lands in a round with no decision file, and D8 escalates it again.
The parent is asked the same question every round until the hard cap, and the accepted findings of the refused round never reach `## Open risks`.

### [MEDIUM] No path gives the next fixer the BLOCKING findings the refusal named

**Location:** `_lyx/discussion/decision-record.md:114`.

**Issue:** The perch runs Bouncer, Burler-Review, Burler-Fix, then Bouncer again.
`BurlerFixProducer.Call` (`internal/shedadapters/burlerfix.go`) fixes round n+1's own accepted review, and round n's fixer ran before round n's judge.
Nothing hands round n's refusal-named findings to round n+1.
Round n+1's focus comes from a judge that converged, so the next reviewer may never raise those findings again.
The mechanism D8 names does not exist.

### [MEDIUM] The Added after Discussion snapshot is lost when the record is retired

**Location:** `_lyx/discussion/decision-record.md:123`.

**Issue:** On the hard-cap continue and on the retired accept at the cap, `retireContinue` moves the escalation record to a timestamped sibling "which no reader opens" (`bouncer.go`, `retireContinue`'s doc).
The next escalation then finds no record and takes a fresh snapshot.
Counterexample: the parent runs `lyx loom decision add`, then records `accept` at the cap.
The refusal retires the record, the round is re-judged, and the re-escalation snapshots titles that already include the parent's entry.
The later accept then carries the finding bare, which defeats D8's "Plan-Write reads a finding beside the parent's answer to it".

### [MEDIUM] D10's no-parent stop defers readable gate behaviour to Plan

**Location:** `_lyx/discussion/decision-record.md:236`, and D10's no-parent bullet at line 150.

**Issue:** The darn writer's one output file is the change description, so the no-parent stop report ends the shuttle run as done.
Darn-Write's first gate is `discussion` with `attempts: 3` (`contracts/recipes/darn-recipe.yaml`, the `Darn-Write` row).
It fails on the absent record and re-prompts the same writer, which by D10 stops again.
After three re-prompts the exhausted gate halts the run "for the parent" (the recipe header), and in this case there is no parent.
The Notes hand this behaviour to the plan, though it can be read today.
That is a deferred research question under the rubric, and a design constraint parked in an optional hints section.

### [LOW] D10's timeout bound names a respawn that does not happen

**Location:** `_lyx/discussion/decision-record.md:154`.

**Issue:** Darn-Write declares no `on_stuck`, so a writer timeout halts the run, and nothing respawns the writer until a resume.
`writer_timeout_min` is also the whole table's deadline (`internal/darnengine/doc.go`, "its wall-clock budget"), so a held wait spends the writer's coding budget.
The bound as written overstates what happens.

### [LOW] D8 leaves open whether a refused accept is a continue on disk

**Location:** `_lyx/discussion/decision-record.md:114`.

**Issue:** `softCapRule` escalates circling only when the previous round's decision is `CirclingContinue`.
If the accept the refusal turns into a continue stays an `accept` on disk, a design-wrong key open across that round and the next escapes the circling check.

## Focus verdicts

- Item 1, accept over a CONVERGED round: the accept goes through the refusal, and its stated bound holds for the accept itself, but the exemption misses the ledger check (F1) and the continue it turns into escapes the bound (F2).
- Item 2, re-entry after a settled accept: holds. `beginSettle` writes the settle record before `settleCirclingAccept`, and `liveSettleRecord` judges liveness by generation and later rounds, not by `Settled`. So a crash after the settle, a commit-seam failure followed by a resume, and an amendment all reach `reenterUnchanged` → `resettle` (closing as accepted) or a new generation, and none reaches the `settleUnconverged` degrade.
- Item 3, read sets against the six-call cap: fits. `goal-scope` spends 1+1+4 calls and `board-fit` 2+4, the same cap `twill-directive.md` gives twill forks; neither lens has a call to spare.
- Item 4, Parent-Align keeping the attempt record: holds. `RenameProducers` rewrites every history producer, so `rowEntry` returns the same index, and D11's fallback read of `Merge-In-fixer-attempts.json` matches its `Entry`.
- Item 5, each wait or stop reaching the parent: the hard-cap re-escalation writes a fresh record and notice, and `arbiterstop`'s default arm refuses the cause and hands back with the notice relayed. The fix row's switch keeps the withholding. The darn writer's wait reaches the parent by `SendMessage` and shuttle's hold notice. The no-parent stop does not halt as stated (F6).

## Rejected

- D3's premise that a strict load refuses an unknown key (raised in passing by bound-breaker, not as a finding): `configengine`'s `RetiredKeys` already drops a retired key at load, and D3's reconcile drop stays correct, so no decision rests on the shaky wording.
