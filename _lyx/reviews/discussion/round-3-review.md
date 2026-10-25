---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D8 (the budget-retire bullet and the escalation brief bullet)
    summary: "D8 says a continue sends the BLOCKING findings to a fixer, but no carrier hands them to the next round's fixer"
    fix: >-
      In D8, replace both claims that a `continue` "sends them to a fixer" (the retire-at-budget sentence and the brief bullet's "a `continue` sends them to a fixer")
      with the mechanism that exists: a `continue` runs another round whose fixer fixes only that round's own review,
      so a listed BLOCKING finding reaches the next fixer only if that round's fresh reviewers re-raise it from the artifact.
      Add that the next round's focus file after a CONVERGED verdict is empty and the decision file carries no text,
      so nothing else carries the findings.
      Then either accept that bound and say so in "What bounds it", or extend the `continue` path so a CONVERGED verdict whose review holds BLOCKING findings takes the refusal's re-judge too,
      so the judge's CONTINUE focus lists them; whichever is chosen, add one acceptance criterion pinning it.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D8 (the hard-cap and budget-retire bullets)
    summary: D8 has no exit when the judgment refusal still holds at or past the hard cap, where only accept is offered and a continue re-escalates
    fix: >-
      In D8's retire-and-re-escalate sentence, limit the re-escalation to rounds below the hard cap.
      State that a refusal still holding at or past the hard cap halts the segment blocked for the operator, as a refused judgment halts at its budget today,
      with a Reason naming the refusal's cause and the way forward (`lyx loom resume` after fixing the artifact or the ledger, or the operator's own halt handling).
      Add an acceptance criterion: a `decision` accept recorded at the hard cap over a judgment the refusal still rejects halts the segment and does not re-escalate.
---

### [MEDIUM] D8 names a fixer carrier that the perch order does not provide

**Location:** `_lyx/discussion/decision-record.md` D8, the sentence "so the parent can record another answer, a `continue` that sends the BLOCKING findings to a fixer among them", and the brief bullet "that a `continue` sends them to a fixer".

**Issue:** The perch runs Bouncer, Burler-Review, Burler-Fix, Bouncer (`contracts/recipes/loom-recipe.yaml`, Discussion rows), so the Bouncer judges round n after round n's fixer already ran on round n's review.
A recorded `continue` returns Stuck into Burler-Review for round n+1, whose fixer fixes only round n+1's own accepted review.
The record's `continue` applies only the dropped-key check, so a `continue` over a CONVERGED verdict is not re-judged.
A CONVERGED judge writes an empty `focus` list (`internal/shedadapters/bouncerprompt.go`, `approved_focus_lists`; `bouncer-template-judge.md`, "A CONVERGED verdict still writes ... focus"), and the circling decision file holds only round, decision, cause, settled and automatic (`internal/shedadapters/circling.go`).
So no row hands a listed BLOCKING finding to the next fixer; it is fixed only if round n+1's fresh reviewers find it again.
The support log's own Rejected alternatives entry says "nothing hands the refusal's findings to the next fixer", which D8's `continue` claim contradicts.
A parent choosing `continue` on the strength of the brief's promise would be misled.
Round 2's F4 raised the missing carrier, and its fixer report closed it with "A BLOCKING finding reaches a fixer only through a `continue`", which names no row that does the handing; the evidence above shows none exists, so the claim does not hold.

### [MEDIUM] A held judgment refusal at the hard cap leaves the parent no exit

**Location:** `_lyx/discussion/decision-record.md` D8, "A refusal that still holds once the segment's budget is reached does not halt there: it retires the decision ... and re-escalates the round with cause `decision`" together with "at or past it, accept only" and "A `continue` that reaches the hard-cap path at settle time re-escalates with cause `decision`".

**Issue:** Round numbers carry on past the budget, so at or past the hard cap the budget is normally reached and no re-judge can be spent.
A refusal that holds there (a CONVERGED verdict over a BLOCKING review finding that is not a `decision` finding, or a dropped ledger key, both of which an accept also checks) retires the accept and re-escalates with only accept offered.
The next accept meets the same refusal, and a recorded `continue` at the hard cap re-escalates too, so neither answer can act and each cycle needs the parent again.
Today a refused judgment halts the segment at its budget (`rejudge`), and a circling accept at the hard cap passes without the refusal, so this loop is new with D8.
"What bounds it" lists the hard cap as a bound but states no terminal state for a refusal that holds at it.

### Settled without a finding

- Both BLOCKING checks of the accept: D8 leaves the round's `decision` findings out of the review check and out of the ledger check by the entry's class, and the acceptance criterion "A BLOCKING `decision` finding alone, whose ledger entry the CONVERGED round holds open, does not refuse the accept" says so. Not a defect.
- Dropped key through a `continue`: D8 applies the dropped-key check to a recorded `continue`, re-judges first, and the acceptance criterion "`continue` recorded on a `decision` escalation over a round whose ledger drops a key ... re-judges" pins it. The record states the bound ("a `continue` starts the next round only over a ledger that keeps every key"). The record has no path in which an accept is turned into a `continue` below the hard cap; a refused accept is re-judged instead. Not a defect.
- What survives of the parent's answer: the circling decision file carries no text, so only an `Added after Discussion` entry survives, and D7 makes an answered question no finding. A bare `continue` without the entry re-raises the finding and re-escalates each round, bounded by "one escalation per judged round" and the hard cap, and the brief's way forward asks for `lyx loom decision add` first. The decision on disk reads as a `continue` to the soft-cap circling check because `softCapRule` reads the previous decision word and never its cause; that is existing behavior the record need not restate. Not a defect.
- The Added-after-Discussion snapshot: it is a separate file of the round, written when absent and moved by neither `retireContinue` nor a re-judge, so a hard-cap or retired-accept re-escalation keeps the first snapshot. A later round snapshots its own titles at its own first `decision` escalation, so the parent's earlier answer is already in it. Not a defect.
- D10 no-parent stop: the record decides it. An absent record fails the darn discussion gate (`discussionparser.Validate` returns a file-missing finding; `internal/loomshed/gates.go` `ValidateDarnDiscussion`), the gate re-prompts up to `attempts: 3`, the stencil has the writer end its turn unchanged, and the exhausted gate halts a row that has no `on_stuck`, as the darn recipe header says. The `writer_timeout_min` bound matches `internal/darnengine/doc.go` and shuttle's held turn end (`internal/shuttleengine/doc.go`). Not a defect.

## Rejected

- A missing path for the snapshot file: the record names the file's role and Plan fixes the path; the plan writer needs no more.
