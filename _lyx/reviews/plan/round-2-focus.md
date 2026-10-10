---
round: 2
focus:
  - "Cards 5 and 6 of the plan, the turn-start guard in `pollEventsTick` (`internal/shuttleengine/wait.go`): does the planned code read the batch's turn starts on every tick that parses events, including a tick whose batch yields no event or ends in a waiting or done event, and lift the guard from that read? A turn-start line yields no event, so settle whether a turn start arriving alone in one tick, and its turn end alone in a later tick, still holds."
  - "Card 5's rule that skips a turn end followed in the batch by a turn start: is it limited to a guard armed at the start of the batch, so that after the guard is lifted, or on a `sessionFakeEngine` run with a zero `PromptOffset` or no `START` before it, a turn end followed by a turn start in one batch still holds as today?"
  - "Card 6's `turnStartRead` and `beginHeldWait`: when one batch holds a held turn end and then a turn start past its offset, does the plan end or never show the held wait while the hold notice still goes, so `held` is never shown beside a `busy` session state? Check that the planned `TestRun_Wait_HeldTurnEnd` rows cover the one-step and two-step shapes."
  - "Card 4's `TestStartup_SkillLoadTurns` row: does the fixture it relies on (`skillStartFixtureWith`, its clock and `skillReed`) offer the mechanism that appends a line to the events file between the last load turn end and the prompt, and is every fixture member the row needs listed under the card's `**Edit:**`?"
  - "Card 3's `observeClocks` first-observation branch and card 2's grep paragraph: does the first observation record the stamp as well as the readings, and does card 2 avoid calling the `Routing.Bounces` doc comment in `internal/shedengine/progress.go` true while it says the Burler row's episode never resets?"
---

Entries target the sites round 1 found defective, restated so the reviewer needs no earlier round.
