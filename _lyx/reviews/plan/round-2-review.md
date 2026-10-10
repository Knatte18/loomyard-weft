---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/06-held-wait-ends-at-turn-start.md:24
    summary: turnStartRead names two reference offsets without saying which governs when a held wait is on show and the tick returns a new held turn end, and no planned row pins that shape
---

### [LOW] `turnStartRead`'s reference offset is ambiguous when a hold is on show and the tick holds again

**Location:** `_lyx/plan/06-held-wait-ends-at-turn-start.md:24` (the `turnStartRead` paragraph) and `:34` (the `TestRun_Wait_HeldTurnEnd` rows).

**Issue:** The card defines `turnStartRead` as a turn start "past heldOffset while a held wait is on show, and past the offset of the held turn end the tick itself returns".
When both apply, a held wait on show and a batch `START`, `STOP:question` that returns a new held turn end, the two clauses disagree: the `START` lies past `heldOffset` but before the new turn end.
Read as "past `heldOffset` whenever a hold is on show", `turnStartRead` is set; `Wait` then ends the old hold through `eventsRead` and, per the card's own rule, skips `beginHeldWait`, so a run whose agent has stopped at a real turn end shows no `held` mark.
Read as "past the latest of the two", it is clear and the new hold shows.
The planned rows cover a hold followed by `START` alone and a `STOP:question` with its `START` in one batch, and the existing multi-stop rows run on `fakeEngine`, which yields no turn starts, so the wrong reading passes every planned test.

**Fix:** State in the `turnStartRead` paragraph that the reference is the tick's own held turn end when the tick returns one, else `heldOffset`, and add a `sessionFakeEngine` row to `TestRun_Wait_HeldTurnEnd`: a `STOP:question` holds, then a `START` and a second `STOP:question` appended in one step leave the marker held, then output plus `STOP:done` finish.

## Focus directives settled

- Cards 5 and 6 read the batch's turn starts right after `ParseEvents` succeeds and `run.offset` advances, before the `len(events) == 0` return in `pollEventsTick` (`internal/shuttleengine/wait.go:640`), so the read runs ahead of both the waiting branch and the `OutcomeDone` return, and a turn start arriving alone lifts the guard; card 5's first `TestRun_Wait_HeldTurnEnd` row pins the `START` alone in one step and its turn end in a later one.
  The Claude parser's `holdFrom` holds back only trailing stamp lines, never a payload line, so a complete turn-start line is always returned in the tick that reads it.
- Card 5's skip rule fires only when the guard was armed at the start of the batch; `armHoldGuard` never arms on a zero `PromptOffset`, a missing capability or no `START` before `PromptOffset`, and card 5's zero-`PromptOffset` row of `TestWait_HeldTurnEndNotice` pins a `STOP:question` followed by `START` in one batch still notifying.
- Card 6: a held turn end with a turn start past it in the same batch sends the notice and skips `beginHeldWait`, and `eventsRead` ends any hold already on show, so `held` never shows beside `busy`; the one-step and two-step rows are both planned. F1 covers the one remaining shape.
- Card 4: `skillStartFixtureWith`'s `configure` hands the test both doubles, `reed.EventsPath` is set before `Start` runs, and `ClassifySkillLoad` is called after the load turn's reader has passed the turn end (`internal/shuttleengine/run.go:617`), so the `OnClassify` hook appends between the last load turn end and the prompt; the members the row needs, `skillFakeEngine` and its `ClassifySkillLoad`, are under `**Edit:**`.
- Card 3's `observeClocks` first observation records the readings and, when known, the stamp; card 2's grep paragraph calls the `Routing.Bounces` clause false and leaves its rewrite to card 1.
