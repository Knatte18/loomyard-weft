# Card 11 — The round after a budget `continue` spends no budget

**Edit:**
- `internal/shedadapters/burler.go`
- `internal/shedadapters/doc.go`
- `internal/shedadapters/burler_test.go`

**Create:**
- `internal/loomrecipe/escalation_test.go`

**Uses:**
- `internal/shedadapters/circling.go`
- `internal/loomrecipe/fixture_test.go`

**Intent:**
D4: a `continue` decision on a budget escalation grants exactly one more round, so the round producer's `Stuck` for that round is `BudgetExempt` as well as the Bouncer's.

In `BurlerProducer.Call` and `probeLiveRound`, the `Stuck` that reports a completed round N+1 is `BudgetExempt` when round N carries a recorded `continue` decision whose cause is `budget`.
Every other `Stuck` stays counted.
A malformed decision file is warned and treated as no exemption.
Bound: the exemption is tied to one decision file for the round before; each further round needs its own decision.
`doc.go`'s BurlerProducer paragraph names the exemption.

Tests:
- `burler_test.go`: the completed-round `Stuck` is exempt after a budget `continue`, counted after a circling `continue`, and counted with no decision.
- `escalation_test.go` drives a real Discussion-Review segment through the loomrecipe fixture with a small `ReviewMaxBounces`, the judge scripted `CONTINUE` every round, and `env.SegmentBounces` wired over the fixture's status file through `Routing(budget)`.
  At the budget the run halts `awaiting`, not `blocked`, with `parent_notice` on the status file.
  After `RecordCirclingDecision` with `continue`, exactly one more round runs without blocking and the next judge escalates again.
  After `accept`, the segment returns `Done`.

**ImpactSummary:** One more budget-exempt `Stuck` path in the round producer, reached only through a recorded budget decision.

**Commit:** 11: The round after a budget continue spends no budget
