---
round: 2
exclude_lenses: [board-fit, goal-scope]
focus:
  - "D4 in `_lym/discussion/decision-record.md` hands the shed driver's arbiter seam (`JudgesFirst`) the stop's reason. The driver builds stops from a persisted status as well as from a step envelope: `stopFromStatus` in `internal/sheddriver/driverules.go` sets `Reason` only for a paused status and carries a blocked halt's text in `Error`, and `internal/sheddriver/drive.go` builds such stops at its start check and on the moved-run recheck. Settle whether the record names the field or fields the seam reads on every path that builds a stop, and whether its acceptance criteria cover a driver that starts or re-checks at a halted landing escalate."
  - "D4 states what a pawl run's fixer escalate does. `internal/loomcli/arbiterprompt.go` lists `lym loom decision add` as a repair verb, `internal/loomcli/decision.go` refuses that verb on a run with no decision record, and `internal/landingshed/fixerrun.go` gives an escalate turn's fixer attempt back. Settle whether the record rests the pawl outcome on a mechanism with cited evidence rather than on an agent's choice, and whether an acceptance criterion pins it."
  - "D3 and D4 in `_lym/discussion/decision-record.md` define a new halt prefix and its readers. `arbiterstop.HandsBack` in `internal/arbiterstop/arbiterstop.go` is true for any reason opening with `shedadapters.DecisionHaltPrefix`, and `landingshed.HandsBack` reads `LandingStopPrefix`. Settle whether the record's constraints on the new prefix make every reader classify it as intended, and whether D4's Bound states what an escalate-resume loop spends, given that the arbiter's per-run spawn cap in `internal/shedarbiter/call.go` is shared with other stops."
---

The two excluded lenses ran in round 1 and found nothing.
The interactions and bound-breaker lenses stay because D3 and D4 change this round.
