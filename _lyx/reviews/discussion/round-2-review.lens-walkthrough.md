```yaml
lens: walkthrough
outcome: reported
```

Task: the walkthrough lens. I ran each decision's mechanism end to end and checked the end state against the decision's stated purpose.

**List: each decision, the case used, purpose met or not**
- D1: delete `internal/alignment`, keep `DecisionNoteTitle` → with parent-review gone, nothing spawns the agent. Met.
- D2: a run waiting in a parent-review round restarts on the new binary → Discussion-Write's gate list is now only `discussion` → it passes. Met.
- D3: a hub `loom.yaml` with `skip_alignment` → a `RetiredKeys` entry drops it at load, and reconcile removes and reports it. Met.
- D4: an archived run with `request.json` and `verdict.json` → `reportengine`'s private parse still counts it. Met.
- D5: `review-discussion` resolves to the four existing lenses plus `goal-scope` and `board-fit`, run as procedures. Met.
- D6, `goal-scope` read count: `board list` (1), `board get` (1), four notes (4) = 6, exactly the cap with none spare. Met.
- D6, `board-fit` read count: `board list` and `peers` (2), then four reads = 6. Met.
- D7: a writer's choice filed as `decision` is ruled out by the rubric's class definition, which reviewers alone enforce. Met as far as discipline goes.
- D8, case A: round 3 CONVERGED, the review holds a LOW `decision` finding, parent answers `accept`, refusal admits → `acceptRound`, the finding is carried, Done. Met.
- D8, case B: round 3 CONVERGED, the review holds `decision` finding D and a BLOCKING `design` finding B, below the hard cap, parent answers `accept` → the refusal rejects over B → the accept runs as a continue → round 4. **Not met**; see F1.
- D8, case C: a round-3 CONTINUE whose ledger drops key K, plus a `decision` finding → escalated before the refusal → parent answers `continue` → `standsAsRecorded` → `settle` without the refusal → round 4 on the defective ledger. **Not met**; see F2.
- D8, case D: a `continue` reaches the hard cap → `retireContinue` → re-escalation with cause `decision` → `Awaiting` with a `ParentNotice` → driver's `EscalationPending` → `arbiterstop` refuses the cause → hand back with the notice relayed. The stop reaches the parent. The `Added after Discussion` snapshot is lost; see F3.
- D8, case E: a crash after `settleCirclingAccept` and before `markSettled`, and a commit-seam failure with resume → `beginSettle` already wrote the settle record, so it is live → `reenterUnchanged` → `resettle` with the accepted closing. Met. An amendment changes the subject → new generation. Met.
- D9: in autonomous mode the writer puts a parent-only question under `## Open risks`, and the perch raises it as `decision`. Met.
- D10, no-parent case: the writer writes the change description as its stop report → its one output exists, so shuttle ends the session as done → Darn-Write's `discussion` gate finds no record → it re-prompts the writer, up to 3 attempts → each re-prompt repeats the stop → the gate is exhausted and the run halts "for the parent", which does not exist. **Not met as stated**; see F4.
- D10, parent case: ask by `SendMessage` and end the turn → shuttle holds the turn and sends a hold notice → the parent's reply wakes the writer → it re-reads the board note. Met.
- D11: a run halted over-cap on `Merge-In` → `RenameProducers` rewrites the history producers → `rowEntry` gives the same index → `Parent-Align` reads `Merge-In-fixer-attempts.json` with a matching `Entry` → `Spent`, `Baseline`, `AlignFrom` and `Halt` are kept. Met.

**Findings**

F1 — MEDIUM, design. `decision-record.md` D8, "Below the hard cap the accept runs as a `continue`".
An accept turned into a continue does not record the parent's answer to the `decision` findings, so the next round's review raises the same `decision` finding again with no decision file. That re-escalates, and the parent is asked the same question every round until the hard cap. Round 3's `decision` findings are also never carried into `## Open risks`, because `acceptRound` never runs for round 3.
Evidence: `internal/shedadapters/bouncer.go:1237-1258` (`acceptRound` alone writes the `CarryOverAccepted` carry-over), `decision-record.md:123` (escalation fires on any round with a `decision` finding and no decision file), `decision-record.md:128`.

F2 — MEDIUM, design. D8, "The escalation runs ahead of the refusal for routing only".
Only an `accept` re-applies `judgmentRefusal`. A `continue` on a `decision` escalation goes through `standsAsRecorded` → `settle`, which skips the refusal. So a judgment the refusal would reject, for example a ledger that drops key K of the previous round, stands as the base of the next round, and K is lost for good. The accept-as-continue path has the same hole: a dropped-key refusal names no BLOCKING finding for the fixer, and the defective ledger is never re-judged.
Evidence: `bouncer.go:491-495` (a recorded round settles without the refusal), `bouncer.go:1012-1016`, `bouncer.go:1048-1065` (dropped-key check), `decision-record.md:126-129`, and D8's bound at `decision-record.md:142-143`, which names nothing a continue lets through.

F3 — LOW, design. D8, "a re-escalation of the round carries that snapshot over".
On the hard-cap path, `retireContinue` moves the round's escalation record aside before `escalate` writes the new one. The archive goes to a timestamped sibling that no reader opens. The re-escalation therefore has no source for the first record's `Added after Discussion` snapshot unless it reads the record before the retire, which D8 does not say. A carried finding would then name the wrong set of entries "recorded since".
Evidence: `bouncer.go:1205-1208`, `bouncer.go:1211-1217`, `decision-record.md:137`.

F4 — MEDIUM, design. D10's no-parent stop, and Notes "the plan pins what Darn-Write's gates do with that state".
The record leaves to Plan a behaviour that can be read today. Darn-Write's `discussion` gate re-prompts the live writer up to 3 times when the record is absent. The writer repeats its stop each time, and then the exhausted gate halts the run "for the parent", which a no-parent run lacks. So the stop costs three re-prompted turns instead of halting at once. This is the deferred research question the rubric flags.
Evidence: `contracts/recipes/darn-recipe.yaml:17-18`, `contracts/recipes/darn-recipe.yaml:68-73`, `decision-record.md:164`, `decision-record.md:250`.

Tool calls: none. Every step above came from the inherited context.
