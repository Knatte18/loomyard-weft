```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal, stop or recovery step a card adds: the next step it names, and whether that step can succeed from the state the refusal leaves.

**Per refusal: next step, progresses or not**
- Card 3, the unknown gate name refusal in `resolvePlainGateClosure`, which names the four legal values. Next step: fix the gate name. Progresses.
- Card 3, a run waiting in a parent-review round under the new binary. Next step: the next gate evaluation passes on `discussion` alone. Progresses.
- Card 4, a run blocked at the old reject cap whose stop text names the removed `lyx loom review approve`. Next step: `lyx loom resume`, as D2 records. Progresses.
- Card 6, a retired key in a hub file. Next step: reconcile drops it through `MigrateConfig`, and run settings are migrated at read time with a Warn. Progresses.
- Card 10, the darn writer with a parent and a plainly outdated entry. Next step: the parent writes a note and replies by `SendMessage`, and the writer re-reads the board. Progresses.
- Card 10, the darn writer with no parent. Next step: a `Decision for &lt;slug&gt;:` note, then `lyx loom resume`. Progresses only when resume spawns a fresh writer; a re-prompted or re-attached writer is not told to re-read the board (finding 2).
- Card 12, the `decision` escalation's Awaiting. Next steps: `lyx loom decision add` (help: refused only while Discussion-Write runs), then `circling continue` or `circling accept`, which `RecordCirclingDecision` admits once cause `decision` is valid, then `lyx loom resume`. Progresses.
- Card 12, a pending accept refused below the budget. Next step: re-judge, then the recorded accept again. Progresses.
- Card 12, a held refusal at the budget below the hard cap. Next step: re-escalate and name `continue`. Progresses when the refusal is a BLOCKING one. It does not progress when it is the dropped-key one, since a `continue` applies the same check (finding 1).
- Card 12, a held refusal at the budget at or past the hard cap. Next step: a `Halt` Stuck naming `lyx loom resume`, and the resume re-judges the round. Progresses, at one judge spawn per resume, as D8 accepts.
- Card 12, a `continue` reaching the hard-cap path at settle time. Next step: `retireContinue`, then re-escalate with accept only. Progresses.
- Card 12, a dropped-key refusal on a recorded `continue`. Next step: re-judge, then Shed's budget halt, then the parent's resume. Progresses as today.
- Card 14, the arbiter refusing a `decision` escalation or a decision halt. Next step: `HandBackWayForward` to the parent, who records the answer or resumes. Progresses.
- Card 14, batten's hand-back seam marking a decision halt. Next step: the orch's resume. Progresses.
- Card 15, a hub `landing.yaml` holding `merge_fix_attempts`. Next step: reconcile renames it, and a document holding both keys keeps the new value. Progresses.
- Card 16, the attempt record decode refusal over the old record. Next step: remove the file named, then `lyx loom resume`, which grants a fresh cap. Progresses.
- Card 16, a run parked on `Merge-In`. Next step: `armAt` renames it to `Parent-Align`, and `HandsBack` still matches the retired name. Progresses.
- Card 16, the rename skipped because a live driver holds the run lock. Next step: the warning, then the next arm renames. Progresses as today.

**Findings**

1. **MEDIUM**, class design, card 12 (Pending accept, the held-refusal branch, and `renderDecisionDetail` with `held` true).
   - Summary: the held-refusal re-escalation names `continue` as the way forward even when the held refusal is the dropped-key check.
     A `continue` applies that same check (card 12, Pending continue), so it re-judges and halts on Shed's budget block instead of starting a round.
   - Evidence: `decisionRefusal(…, CirclingAccept)` runs `judgmentRefusal`'s whole check, which includes the dropped-key check for round 2 or later (`internal/shedadapters/bouncer.go:1053-1066`).
     The card's continue branch applies only that check.
     Card 12's test row 8 covers only a BLOCKING `design` refusal.
   - Fix: when the held refusal is a dropped-key one, the detail says that a `continue` meets the same refusal and halts the segment until a resume re-judges it.
     Test row 8 gains that case.

2. **LOW**, class consistency, card 10 (the no-parent stop bullet).
   - Summary: the way forward is a decision note, then `lyx loom resume`.
     The stencil tells the writer to re-read the board only "on any message from your parent".
     A writer with no parent that a resume re-attaches to, or the gate re-prompts, is told to end its turn unchanged "while that stop still holds", with no step that checks for the new note.
     So that resume can repeat the stop.
   - Evidence: `_lyx/plan/10-darn-writer-entry-check.md`, the bullets "On any message from your parent, re-read the board for the note" and "When the handoff's `discussion` gate re-prompts you while that stop still holds…".
     D10 (decision-record.md:164) says a re-attached writer gets a fresh deadline and re-reads the board on a parent message.
   - Fix: the stencil tells a re-prompted or resumed writer with no parent to re-read the board for a `Decision for &lt;slug&gt;:` note before it keeps the stop.

Tool calls:
1. Grepped `bouncer.go` past `judgeCall` for re-judge context, and `shuttleengine/gate*.go` for persisted gate-state keying, to check whether a re-judge or a removed gate name changes the next step. The manifest held neither.
2. Read `lyx loom decision add --help` and grepped `shuttleengine` for `GateAttempts`, to check that `decision add` succeeds while the run awaits at Discussion-Bouncer. The manifest lacked the verb's refusals.
