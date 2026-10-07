---
round: 3
focus:
  - "Plan card for the turn-end hold (`_lyx/plan/05-hold-turn-end.md`): after Wait stops returning `OutcomeAsking`, does the card state a rule, not a name list, covering every existing test in `internal/shuttleengine` (including `attach_test.go`, `wait_test.go` and `waiting_test.go`) that expects `OutcomeAsking` from `Wait` or `pollEventsTick`, and does every such test's file appear among the card's Edit targets or a card that lands at or before the commit where it breaks?"
  - "Shed driver stencil card (`_lyx/plan/13-tail-in-stencils-and-steer.md`): `contracts/stencils/shed/shed-template-driver.md` section `## Notifying the parent` says the generic escalation is a SendMessage naming the run-id and stop report path 'and nothing more', that a `parent_notice` is sent 'verbatim', and that the notice is the whole message; does the card reconcile every one of those sentences with the tail it adds, so the section states one rule?"
  - "Public clock seam (`Clock` type and `Runner.SetClock` in `internal/shuttleengine`, introduced by `_lyx/plan/14-replay-corpus.md`): is the seam a separately reviewable unit with its own `Create` handle, and does the Intent state what it lets a caller skip and what bounds it?"
  - "ImpactSummary in `_lyx/plan/05-hold-turn-end.md`: does it pin a count of packages or other perishable tally for the `AwaitOperator` assignments rather than naming the source?"
---

Focus entries restate the sites they depend on and cap no severity.
