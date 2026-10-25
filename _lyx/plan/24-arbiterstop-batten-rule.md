# Card 24 — arbiterstop-batten-rule

**Edit:**
- `internal/arbiterstop#Stop`
- `internal/arbiterstop#Eligibility`
- `internal/arbiterstop#Readers`
- `internal/arbiterstop#Check`
- `internal/arbiterstop/doc.go`

**Create:**
- `plan:internal/arbiterstop#NewBattenReaders` -> `func NewBattenReaders() Readers`
- `plan:internal/arbiterstop#BattenHandBackWayForward` -> `const BattenHandBackWayForward = ""`
- `plan:internal/arbiterstop#TestEligibility_BattenRule` -> `func TestEligibility_BattenRule(t *testing.T)`

**Intent:**
Give the arbiter's eligibility a rule per recipe (D8): for batten, a stop the tackler escalated is eligible unless it is a hand-back, which goes to the run's parent as today; loom's and darn's rules are unchanged.
`Readers` gains `Batten bool`, and `NewBattenReaders` returns readers with it set, no pending-escalation read and the hand-back interrupt policy for every row.
`Stop` gains `Batten`, `BudgetExhausted` and `PreconditionUnmet`; `Check` fills the last two from the status error's `shedengine.ReasonBounceBudgetExhausted` and `shedengine.ReasonPreconditionUnmet` prefixes.
`Eligibility` with `Batten` set refuses a landing halt, a spent budget and an unmet precondition, each naming the hand-back with `BattenHandBackWayForward`, a new clause naming `lyx batten status <slug>` where loom's names `lyx loom status <run-id>`, and admits every other stop, error stops of any kind and blocked stops included; without it the loom rule runs exactly as today.
The driver's rule table hands those three back before the tackler, so no driven stop reaches the batten refusals: they are a fail-loud guard in `refusal-spec.md`'s sense against a recipe wiring that sends one to the arbiter, and the test pins the rule through `Check`.
The package doc states the per-recipe rule.
`TestEligibility_BattenRule` (tier0, parallel) rows through `Check` with `NewBattenReaders`: a blocked Land stop with a deploy error is eligible, a failed Worktree-Teardown stop of kind producer is eligible, a blocked stop whose error starts with the budget prefix, one with the precondition prefix and a landing halt are refused with `BattenHandBackWayForward`; through loom-shaped readers, only the two statuses `TestCheck_RefusesLandingHalts` does not pin, the budget-prefix and precondition-prefix blocked stops, keep loom's answer (eligible, as any blocked stop).

**ImpactSummary:** One flag on the readers selects batten's rule; every existing caller builds loom readers and reads the same answers.
