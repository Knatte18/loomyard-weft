# The sampler carries drift rows, off by default and switched on per task

A `sampler` run could not reach drift: every `lym drift` verb runs only in a tapestry or darn run's worktree, and the sampler recipe had no drift rows.
Drift's own stages could therefore be tried for the first time only on a production task.
The sampler recipe now carries the drift rows, so the operator can try the whole drift flow on a deliberate change first.

Closes #28

## What changes

- `contracts/recipes/sampler-recipe.yaml` gets darn's drift rows, with the same names, engines and config:
  `Drift-Baseline`, `Drift-Await` and `Drift-Expect` before `Sampler-Session`, `Drift-Verify` after it, and `Drift-Recheck` after the Code-Review perch.
- Each row carries the entry gate `drift.enabled` and `drift.scope == full`.
  A task switches them on with `{"drift": {"scope": "full"}}` in its `run_settings`, in a hub that has `drift.enabled` on.
  With `drift.scope` at its default `""` or at `off`, each row is recorded skipped and a sampler runs as before.
- The `lym drift` verbs now serve a sampler worktree whose drift rows are on.
  The refusal messages in `internal/driftcli` and the verb help name sampler beside tapestry and darn.
- A Stuck of `Drift-Verify` or `Drift-Recheck` in a sampler run names `lym loom goto --to Sampler-Session` then `lym loom resume` as the route that fixes code.
  `fixRoute` in `internal/driftengine/verify.go` derives it from the run's rows: `Darn-Write`, else `Sampler-Session`, else `Webster`.
- `docs/overview.md`, the `internal/loomrecipe`, `internal/shedrecipe` and `internal/driftengine` package docs and the `drift.yaml` template comments describe the sampler's rows.

## Design decisions

- The switch is the existing `drift.scope` of the task's `run_settings`, not a new key.
  Its default `""` leaves the sampler's rows out, where tapestry and darn treat `""` as `full`.
  The gate is on the rows, so the `drift.scope` semantics in `Drift-Baseline` are unchanged.
- The stage gates of tapestry and darn are unchanged.
  The `Sampler-Session` row carries no drift gate: the operator runs `lym drift check` on demand, and `Drift-Verify` is the check.
- The drift rows carry no `on_stuck`, like darn's: a Stuck of one halts the run for the parent with the ways forward in its reason.
- `internal/loomcli/wiring.go` now tells the Drift expect agent a plan directory only when the recipe has a `Plan-Write` row, where it excluded only darn.
  A sampler run has no plan and is told so, as a darn run is.
  `TestWire_DriftExpectPlanReading` pins this per recipe.

## Tests

- `TestDriftRows_EntryGate` covers every drift row of the three recipes over `drift.enabled` and `drift.scope` values.
- `TestSamplerRecipe` pins the new row order.
- `TestDriftVerify` gains a case for the route under the sampler's rows.
- `TestWire_SamplerRecipe` checks the session stamp after a skipped `Drift-Expect`.

## To check by hand

- No sampler run with `drift.scope` set to `full` has been run end to end against a real hub with drift configured.
  Run one on a deliberate change to see the baseline, the sealed expectation, `Drift-Verify` and `Drift-Recheck` work in a sampler worktree, and `lym drift check` and `lym drift amend` from its worktree.
