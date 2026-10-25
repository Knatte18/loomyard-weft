# Round 3 fixer report — discussion

## Changes

- F1 (MEDIUM), fixed as proposed, taking the "accept the bound" branch.
  Premise verified: the Discussion ring runs Bouncer, Burler-Review, Burler-Fix and back to Bouncer (`contracts/recipes/loom-recipe.yaml`), a CONVERGED judge writes an empty `focus` (`internal/shedadapters/bouncerprompt.go`, `approved_focus_lists`), and the circling decision file holds only its YAML header (`internal/shedadapters/circling.go`, `circlingDecisionHeader`).
  In D8, the retire-and-re-escalate sentence now says "a `continue` among them", and the brief bullet says a `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.
  "What bounds it" now states that no carrier exists: the round's own fixer ran before the judge, the next fixer fixes only its own review, and the focus file and decision file carry nothing.
  It also states why the bound is accepted: the round's own fixer already had every finding except the `decision` ones.
  The re-escalation criterion now pins it: the `continue` runs another round with an empty focus file, and that round's fixer gets only its own review.
- F2 (MEDIUM), fixed differently from the proposal on one point.
  D8 now limits retire-and-re-escalate to rounds below the hard cap.
  At or past the hard cap, a refusal still held at the budget halts the run with a Reason that names the refusal's cause and the way forward, `lyx loom resume`.
  A new acceptance criterion pins that the halt happens and the round does not re-escalate.
  The proposal said the halt works "as a refused judgment halts at its budget today".
  I did not do that, because a budget halt persists the shed-wide `ReasonBounceBudgetExhausted` reason "regardless of the producer's Reason" (`internal/shedengine/doc.go`; `internal/shedengine/run.go`, the budget-capped persist), so the Reason could not carry the cause.
  D8 instead blocks the run at the Bouncer's row with a budget-exempt `OutputPointer.Halt` Stuck, which keeps the producer's Reason (`internal/shedengine/doc.go`, "A Stuck verdict whose OutputPointer.Halt is set").
  I also dropped the proposal's "or the operator's own halt handling", because it names no step to carry out (see Self-check).
- Self-check follow-ups, made in D8 and Constraints:
  - D8's brief bullet tells the parent that an accept at or past the hard cap halts the run if the refusal still holds at the budget.
  - The way-forward clause requires the re-judged ledger to keep every key of the previous round's ledger.
  - The record says that a re-judge which is refused again halts again.
  - `PATTERN-refusal-way-forward` in Constraints now names the hard-cap halt.

## Self-check

- fix-consistency, LOW: Constraints did not name the new halt Reason. Fixed as proposed.
- fix-consistency, LOW: the brief bullet did not warn that an accept can halt the run at the hard cap. Fixed as proposed.
- fix-claims, MEDIUM: "a judge that now rules CONTINUE or CIRCLING lets the accept pass" is false for a dropped-key refusal. Fixed as proposed: the clause now requires the ledger to keep every key of the previous round's ledger.
- fix-proposal: no findings.
- fix-way-forward, MEDIUM: a budget halt drops the producer's Reason, so the cause never reaches the stop. Fixed as proposed, through the `OutputPointer.Halt` branch it named. The criterion now reads "blocks the run at Discussion-Bouncer with a budget-exempt halt", and D8 says how a resume re-judges.
- fix-way-forward, LOW: "or the operator's own halt handling" names no step. Fixed as proposed: the clause is dropped, and D8 says that a re-judge which is refused again halts again, with one judge spawn per resume.

## Disputed

None.

## Deferred

None.
