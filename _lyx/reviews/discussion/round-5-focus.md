---
round: 5
focus:
  - "In `_lyx/discussion/decision-record.md` D4, settle whether the budget-reached test is stated exactly once, as count >= budget, with count the Bouncer row's own counted `Stuck`s before the current round's `Stuck` (seed included) and budget the effective budget Shed's engine applies to that row; check against `internal/shedengine/progress.go` (`Routing.Bounces`) and `internal/loomcli/wiring.go` (`segmentBounces`) that the budget the seam reports is the same config-applied `review_max_bounces` the engine uses once D3 removes the literal `max_bounces` from the review rows."
  - "In `_lyx/discussion/decision-record.md` D4, its Open risks and its Acceptance criteria, settle whether the claim that a `continue` on a circling escalation is counted and never blocked is conditioned on the `Bounces` seam returning `ok` true, and whether the residual halt to Shed's generic `bounce-budget-exhausted` block is named for every unwired, `ok` false or errored read."
  - "In `_lyx/discussion/decision-record.md` D8 and D10, settle whether the Bound paragraph states only what is provided (entries marked and committed, no reviewer checking them, a Plan-Write addition licensed by Plan-Review without a second reader) and does not call visibility a bound."
---

Each entry names a Decision and the question to settle, and cites the code sites it depends on.
