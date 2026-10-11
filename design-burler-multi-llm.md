# Burler runs its round on MultiLLMProducer

`burler-multi-llm` · Task · enhancement, burler, shed, stencil · running · Webster-Bouncer

## Goal

Burler's review round runs on `MultiLLMProducer` instead of its own two-agent wiring (operator, 2026-10-08).

## Today

Burler's round has run two agents ad hoc since 572ab84e8: `roundHalves` in `internal/burlerengine/engine.go` starts the reviewer and the fixer, and the fixer waits on the review through `lyx burler await-review`.
The round wires six stencils (two orchestrator templates, three step files, a focus directive) plus per-segment rubrics through Go code spread over several packages.

## Wanted

- The fixer is the `chair` and the reviewer `advisor-1`, declared in one seat table.
- The ad hoc wiring (`roundHalves`, `await-review`) goes; the round's outcome, review files and Bouncer contract stay as they are.
- Fanned rounds (`discussion_fan`, `plan_fan`) keep working; whether a fan's lenses become advisor seats or stay forks of the reviewer is decided in Discussion.

## Required (operator, 2026-10-11)

- **Warm cache.** Today the fixer waits through `lyx burler await-review`, capped under the 5-minute prompt-cache lifetime so every re-run keeps its cache warm (`internal/burlercli/await.go:25`).
  A seats chair instead ends its turn and waits idle, so a review longer than 5 minutes would wake the fixer on a cold cache every round.
  The chair keeps its cache warm at least as well as today, using the general capped wait command that `twill-discusser` builds, not a Burler-specific one.
- **No false stall notices.** Folded in from `seats-chair-wait-notice` below: with the fixer as chair, every Burler round of every run would otherwise send the parent stall notices.

## Seats chair waits reach the parent as stall notices (was `seats-chair-wait-notice`)

### Seen (2026-10-11, discussion bench)

Every time a seats chair ends its turn to wait, for an advisor's up line or for an advisor's answers, shuttle sends the parent a "turn end held, output files missing" notice (`internal/shuttleengine/hold.go:53`, `notifyHeld`).
These waits are planned by the chair stencil, so the notices are false stall alarms; each advisor run in the bench sent the orch three to four of them.
Advisors are quiet because `internal/seatengine/prompt.go:160` sets `QuietHold: !isChair`; the chair is not.
Answering a notice wakes the chair for a turn, which sends another notice.

### Idea

Suppress the held notice for a chair while any advisor has an unanswered batch or has not yet sent its up line, and keep it for a chair that ends a turn with no advisor work outstanding, which is a real stall.
