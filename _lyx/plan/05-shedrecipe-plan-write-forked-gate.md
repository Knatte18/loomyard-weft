# Card 5 — shedrecipe-plan-write-forked-gate

**Create:**
- `internal/shedrecipe#planWriteGateSpec`
- `internal/shedrecipe#noFindingsPlanIndex`
- `internal/shedrecipe#noFindingsPlanIndex.ValidateFormat`

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

`fixture_test.go` gains `noFindingsPlanIndex`, a test index that embeds `planindex.Index` and answers no findings and a nil error from `ValidateFormat`, the one index method `ValidatePlan` calls when the anchor holds no webster record.
`TestPlanWriteEntry_GateConfig` gains the subtest "PlanWriteSetsForkedOnItsPlanEntry": over `newTestEnv(t)` with `env.PlanIndex` set to `noFindingsPlanIndex{}`, and `gatesCfg("plan", 3)`, it seeds a minimal parseable plan (`00-overview.md` plus one card file) under `planparser.PlanDir(env.AnchorPath)` with no `lens-review.md`.
`planWriteGateSpec` returns one entry whose `Gate` and `Forked` are both non-nil; `Gate()` passes, while `Forked()` fails with findings naming `lens-review-missing`, so the row's `Forked` is the lens-review closure and not the plain plan gate.
The lens-review verdict's other cases are pinned by card 4's `TestNewPlanGate` rows and its selection by card 3's `TestAttach_GateThreading` rows.

**ImpactSummary:** one private helper replacing one call inside `planWriteEntry`; no other entry, recipe key or exported signature changes.

**Commit:** 5: shedrecipe: Plan-Write's plan gate carries the lens-review closure for a forked run
