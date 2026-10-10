# Plan-Review round 1 — fixer report

Every finding was validated against the code and the decision record, and every one was fixed.

## Fixed

- **F1 (BLOCKING)** — confirmed: `pollEventsTick` returns through `expiredTurnEnd` at `len(events) == 0` (`internal/shuttleengine/wait.go:640`), and a turn start yields no event.
  Card 5 now computes `turnStartOffsets` right after `ParseEvents` succeeds and `run.offset` advances, before that return, and lifts the guard on any turn start whatever the batch's events.
  Card 5's AC10 row now appends the prompt's `START` alone in one step and `STOP:question` in a later step, asserting the later step holds.
  Card 6 states that `turnStartRead` is set from that same early read.
- **F2 (BLOCKING)** — confirmed.
  Card 5's rule now skips a turn end only when the guard was armed at the start of the batch and no turn start of the batch precedes the turn end's line; with the guard off at batch start, the batch's turn starts change no hold.
  Card 5's `TestWait_HeldTurnEndNotice` gains a `sessionFakeEngine` row with a zero `PromptOffset` and `STOP:question` then `START` in one step, asserting the notice goes with the `STOP` line's `NotifiedOffset`.
- **F3 (MEDIUM)** — confirmed: `Wait` calls `endHeldWait` before the held branch calls `beginHeldWait` (`wait.go:250`–`271`).
  Card 6's `turnStartRead` now also fires on a turn start past the offset of the held turn end the tick itself returns.
  When it is set, `Wait` sends the notice as today and does not call `beginHeldWait`.
  Card 6 adds a `TestRun_Wait_HeldTurnEnd` row with `STOP:question` and `START` in one step, asserting the marker reads not held at the next step.
  That shape's hold notice is pinned by card 5's F2 row.
  Card 6's ImpactSummary now counts two conditions in `Wait`.
- **F4 (MEDIUM)** — confirmed: `skillStartFixtureWith` builds a plain `fakeClock`, and neither it nor `skillReed` has a step hook.
  Card 4 now edits `skillFakeEngine` and `skillFakeEngine.ClassifySkillLoad`, which gain an `OnClassify` hook.
  The load turn's reader advances `run.offset` (`run.go:651`) before it classifies (`run.go:617`), so a line the hook appends lies after the last load turn end and before the prompt.
  The test row passes the hook through `skillStartFixtureWith`'s existing `configure` parameter.
- **F5 (LOW)** — confirmed.
  Card 3's `observeClocks` first-observation branch now records the stamp, when known, beside the wall and awake readings.
- **F6 (LOW)** — confirmed: `internal/shedengine/progress.go:102` says the Burler episode never resets, against `episodeStuckCount`'s doc and D1.
  Card 1 now edits `internal/shedengine#Routing.Bounces`, comment only, with its code unchanged.
  The rewritten comment drops that clause and says an off-line Burler-round row reports its step's main-line row's count.
  Card 1's ImpactSummary now covers this edit.
  Card 2's grep sentence now says every Go hit for "never resets" sits in a doc comment card 1 rewrites, and no longer calls the `progress.go` comment true.

## Disputed

None.

## Deferred

None.
