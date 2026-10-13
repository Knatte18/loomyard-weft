# Card 5 — shedrecipe-plan-write-forked-gate

**Create:**
- `plan:internal/shedrecipe#planWriteGateSpec` -> `func planWriteGateSpec(cfg Config, env Env) (shuttleengine.GateSpec, error)`

**Edit:**
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe#TestPlanWriteEntry_GateConfig`

**Intent:**
D3's wiring: only the Plan-Write row's `plan` gate entry carries the lens-review closure, so a twill-spawned Plan-Write run gets the lens-review check while Plan-Burler-Fix (`gates: [plan]` too), PR-Rework (`rework-plan`) and a non-twill Plan-Write keep today's check set.

`planWriteGateSpec` resolves the row's `gates` key through `resolveGateSpec("PlanWrite", cfg, env)`, then sets `Forked` on every resolved entry named `plan` to `loomshed.NewLensReviewPlanGate(env.AnchorPath, env.WorktreeRoot, env.RunScratchDir, env.PlanIndex)`.
The `plan` name's resolution has already required those roots and the index, so the closure is built over checked values.
`planWriteEntry` calls it in place of its direct `resolveGateSpec` call.
Its doc comment drops the claim that the entry never makes a validator choice of its own: the `gates` key still names the validator, and the entry adds the lens-review closure that shuttle evaluates in place of the `plan` closure for a run whose record carries fork authorization (card 3), which `PlanSpec` sets exactly under `plan_producer: twill`.
Selecting by the row's own name in `resolveGateSpec` stays rejected; the dedicated constructor is what tells the Plan-Write row apart, so `resolveGateSpec`, which every other gated entry calls, never sets `Forked`.
Bound: `Forked` is consulted only for a fork-authorized run of this row, and it runs the plan gate's own set plus one presence check.

`TestPlanWriteEntry_GateConfig` gains the subtest "PlanWriteSetsForkedOnItsPlanEntry": over `newTestEnv(t)` and `gatesCfg("plan", 3)`, `planWriteGateSpec` returns one entry whose `Gate` and `Forked` are both non-nil, and driving `Forked()` over `newTestEnv`'s empty anchor returns a failed result whose findings name the missing plan overview, as the existing subtest drives `Gate`, proving `Forked` is a plan-gate closure.
The lens-review half of its verdict is pinned by card 4's `TestNewPlanGate` rows and its selection by card 3's `TestAttach_GateThreading` rows; `newTestEnv`'s `placeholderIndex` cannot validate a plan, so this subtest stops at the parse.

**ImpactSummary:** one private helper replacing one call inside `planWriteEntry`; no other entry, recipe key or exported signature changes.

**Commit:** 5: shedrecipe: Plan-Write's plan gate carries the lens-review closure for a forked run
