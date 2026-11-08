# The sampler carries drift rows, off by default and switched on per task

A sampler run could not reach drift, so Drift-Baseline, Drift-Expect, the drift check, the judge and `lym drift amend` could first be tried only on a production task.
The sampler recipe now carries darn's Drift rows, off by default, so the operator can run the whole drift flow on a deliberate change.

## What changes

- `contracts/recipes/sampler-recipe.yaml` gains Drift-Baseline, Drift-Await and Drift-Expect between Record-Write and Sampler-Session, Drift-Verify between Sampler-Session and the Code-Review perch, and Drift-Recheck between the perch and Describe.
  The rows take darn's names, engines and config.
- Each sampler Drift row carries the entry gate `drift.enabled` and `drift.scope == full`.
  `drift.scope` defaults to `""`, so a sampler records every Drift row skipped and runs as before.
  A task switches the rows on with `{"drift": {"scope": "full"}}` in its `run_settings`; the hub must also have drift enabled, as for darn.
  No new config key is added.
- Drift-Verify's way forward routes the code fix to `Sampler-Session` when the run's rows hold it (`internal/driftengine/verify.go`), in place of the `Webster` route a sampler would otherwise get.
- The expect agent is told the plan directory only when the recipe has a Plan-Write row (`internal/loomcli/wiring.go`), so a sampler's agent is told no plan, as darn's is not.
- The `lym drift` verbs' help and refusals name sampler runs beside tapestry and darn.
  The verbs already resolve any run's worktree with a seed and status, so a sampler worktree with the rows on is served unchanged.
- The tapestry and darn recipes and their stage gates are unchanged.

## Design decisions

- The rows need both `drift.enabled` and `drift.scope == full`.
  The producers pass at once while `drift.enabled` is false, so gating on the scope alone would enter rows that do nothing.
- Sampler-Session gets no `drift` gate: it is an operator session with no gates.
  The operator runs `lym drift check` on demand, and Drift-Verify and Drift-Recheck are the check.
- `lym drift amend` and `lym drift defer` keep their admitted rows: a sampler's drift halts land on Drift-Verify or Drift-Recheck, which both verbs already admit.

## Tests

- `TestDriftRows_EntryGate` covers the sampler: its rows hold only with `drift.enabled` true and `drift.scope` full, and tapestry's and darn's ignore the scope.
- `TestSamplerRecipe` pins the new on_done path.
- The Drift-Verify table gains a sampler-rows case naming `lym loom goto --to Sampler-Session`.
