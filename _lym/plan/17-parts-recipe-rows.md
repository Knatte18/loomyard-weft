# Card 17 — parts-recipe-rows

**Create:**
- `plan:internal/loomshed#NamePartsWrite` -> `const NamePartsWrite = "Parts-Write"`
- `plan:internal/loomshed#NamePartsPlanWrite` -> `const NamePartsPlanWrite = "Parts-Plan-Write"`
- `plan:internal/loomrecipe#TestPlanSplitRows` -> `func TestPlanSplitRows(t *testing.T)`
- `plan:internal/loomrecipe#TestSequence_PlanSplitRunsThePartsRows` -> `func TestSequence_PlanSplitRunsThePartsRows(t *testing.T)`

**Edit:**
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/loomshed#InterruptPolicies`
- `internal/loomshed#TestInterruptPolicyFor`
- `internal/reportengine#loomPhases`
- `internal/reportengine#loomRowPhases`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#deriveSequenceOrder`
- `internal/loomrecipe#TestEntryNumbers`
- `internal/loomrecipe#gatedRowFanViolations`
- `internal/loomrecipe#writeLoomOutputs`
- `internal/loomrecipe/doc.go`
- `docs/overview.md`
- `internal/reportengine/doc.go`

**Uses:**
- `internal/loomshed/doc.go`
- `internal/loomrecipe#TestDriftRows_EntryGate`
- `internal/shedengine#TestGoto_StepResumesAtTarget`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/loomrecipe#TestSequence_FullRunBlocksAtMergeIn`

**Intent:** Put the two rows into the tapestry recipe (D1, D13), switched by `tapestry.plan_split`, and give them the plumbing every tapestry row has.

**Recipe.** In `contracts/recipes/tapestry-recipe.yaml`, `Drift-Baseline`'s `on_done` becomes `Parts-Write`, and two rows go between it and `Plan-Write`:

```yaml
  # Parts-Write and Parts-Plan-Write write the plan in two steps when tapestry.plan_split is true: Parts-Write splits
  # the task into parts with contracts, and Parts-Plan-Write runs one writer per part and a chair that joins their cards
  # into one ordinary plan. With the flag off both rows are recorded Skipped and Plan-Write runs as before; with it on,
  # Parts-Plan-Write's Done enters Plan-Write, whose gate skips it to Plan-Bouncer. Neither carries on_stuck: a Stuck
  # halts the run as Plan-Write's does, and the way back is a rerun of the halted row.
  - name: Parts-Write
    engine: PartsWrite
    entry_gate: tapestry.plan_split
    on_done: Parts-Plan-Write
    config:
      gates:
        - name: parts
          attempts: 3

  - name: Parts-Plan-Write
    engine: PartsPlanWrite
    entry_gate: tapestry.plan_split
    on_done: Plan-Write
    config:
      # gates holds the chair's handoff until the joined plan passes the plan gate; part_gates gives each writer seat
      # the plan-part gate over its own part directory.
      gates:
        - name: plan
          attempts: 3
      part_gates:
        - name: plan-part
          attempts: 3
```

`Plan-Write` gains `entry_gate: tapestry.plan_split != true` and keeps `on_done: Plan-Bouncer`.
The header comment's main line gains `Parts-Write` and `Parts-Plan-Write` before `Plan-Write`, their switch is described beside the Drift rows' entry gate, and the list of producer rows that carry no `on_stuck` gains both.
Bound, stated in that comment: each row reads the flag when the run enters it and a goto bypasses only its target's gate, so a flag flipped mid-branch routes the rest of the run by the new value; the flag is meant to be set before a run starts, in its profile or `run_settings`.

**Plumbing.** `NamePartsWrite` and `NamePartsPlanWrite` join loomshed's row names; `InterruptPolicies` maps both to `shedverbs.InterruptPolicyReinvoke`, with a comment that each probes for a live chair and resumes it through the seat probe; `loomPhases` maps both to `PhasePlan`, so a bench reads one plan-phase span whichever producer ran.
`internal/reportengine/doc.go`'s list of a tapestry run's plan-phase rows ("plan every Plan row, Drift-Baseline and Drift-Await") names `Parts-Write` and `Parts-Plan-Write`.
`TestInterruptPolicyFor` gains a row for each, and `loomRowPhases`, the timing test's list of every loom row and its phase, gains both with `PhasePlan`.
`gatedRowFanViolations` admits `PartsWrite` and `PartsPlanWrite` among the writer engines whose gated rows may carry no fan, and its refusal message, which lists the writer engines, names both.
`internal/loomrecipe/doc.go`'s sentence that `Drift-Baseline` sits between the Discussion-Review ring and `Plan-Write` says it sits before `Parts-Write`, with the two parts rows between it and `Plan-Write`.
`wantProducerTable` gains both rows between `Drift-Baseline` and `Plan-Write`, each typed `reflect.TypeOf(loomshed.NewPlanWrite("", nil, nil))`, which pins both names through the coverage guard; `deriveSequenceOrder` records both as `shedengine.OutcomeSkipped`, since the kit's template answers `plan_split` false.
`TestEntryNumbers` renumbers the loom rows: `Parts-Write` 5, `Parts-Plan-Write` 6, `Plan-Write` 7 and every later row two higher.

`TestPlanSplitRows` (tier0, built as `TestDriftRows_EntryGate` is) covers, for `plan_split` false and true: both new rows' entry gates hold exactly when the flag is true and `Plan-Write`'s exactly when it is false; the edges `Drift-Baseline` → `Parts-Write` → `Parts-Plan-Write` → `Plan-Write` → `Plan-Bouncer`; neither new row has an `on_stuck`.
With the flag off the sequence run records both new rows Skipped and runs `Plan-Write`, which the sequence test pins.

`TestSequence_PlanSplitRunsThePartsRows` (tier0) drives the flag-on run over `buildSequenceFixture` with `env.Setting` answering `tapestry.plan_split` true.
`writeLoomOutputs` gains two role branches for it: the `parts` role writes a parts file of one part, named `whole`, carrying every decision id of the fixture's record, and the `parts-plan` chair writes the part's overview, the plan directory as the `plan` role does, and the join record.
It asserts the history runs `Parts-Write` Done, `Parts-Plan-Write` Done and `Plan-Write` Skipped, then enters `Plan-Bouncer`.
It then moves the run onto `Plan-Write` with the built shed's goto and steps it once: `Plan-Write`'s producer runs and records Done although its entry gate does not hold, since a goto bypasses only its target's gate.

`docs/overview.md`'s loom entries name the two rows, their switch and their gates, and its regression-rows sentence says `Drift-Baseline` comes before `Parts-Write` in loom.

**ImpactSummary:** Two recipe rows, Plan-Write's entry gate, two names with policy and phase, and the loomrecipe and timing pins; no edge out of Plan-Write changes.
