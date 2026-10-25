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
`Stop` gains `Batten`, `BudgetExhausted`, `PreconditionUnmet` and `PauseHonoured`; `Check` fills the middle two from the status error's `shedengine.ReasonBounceBudgetExhausted` and `shedengine.ReasonPreconditionUnmet` prefixes, and `PauseHonoured` for a blocked Land stop whose reason starts with card 18's `battenshed.PauseHonouredPrefix`.
`Eligibility` with `Batten` set refuses a spent budget, an unmet precondition and a pause-honoured Land stop, each naming the hand-back with `BattenHandBackWayForward`, a new clause naming `lyx batten status <slug>` where loom's names `lyx loom status <run-id>`, and admits every other stop, error stops of any kind and blocked stops included; without it the loom rule runs exactly as today.
Batten has no landing halt: `Check` computes `LandingHalt` through `landingshed.HandsBack`, which knows only Merge-In and Finalize, and no batten row has either name, so no batten stop is one.
The driver's rule table hands the spent budget and the unmet precondition back before the tackler, so no driven stop reaches those two refusals: they are a fail-loud guard in `refusal-spec.md`'s sense against a recipe wiring that sends one to the arbiter, and the test pins the rule through `Check`.
A pause-honoured Land stop persists blocked and reaches the tackler and then the arbiter, so its refusal is the one a driven stop meets: under the Added-after-Discussion entry "A pause-honoured Land stop is a hand-back under D8" (parent, 2026-10-25) the arbiter never relaunches over an operator's or tender's pause, and the stop goes to the run's parent, where card 18's router sends no notice for it.
D8's batten landing halt, which the acceptance criteria keep from the arbiter, is this Land stop handed back; `LandingHalt` itself stays loom's.
The package doc states the per-recipe rule.
`TestEligibility_BattenRule` (tier0, parallel) rows through `Check` with `NewBattenReaders`: a blocked Land stop with a deploy error is eligible, a failed Worktree-Teardown stop of kind producer is eligible, a blocked stop whose error starts with the budget prefix, one with the precondition prefix and a blocked Land stop whose reason starts with `battenshed.PauseHonouredPrefix` are refused with `BattenHandBackWayForward`; through loom-shaped readers, only the two statuses `TestCheck_RefusesLandingHalts` does not pin, the budget-prefix and precondition-prefix blocked stops, keep loom's answer (eligible, as any blocked stop).

**ImpactSummary:** One flag on the readers selects batten's rule; every existing caller builds loom readers and reads the same answers.
