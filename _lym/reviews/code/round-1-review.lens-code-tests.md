```yaml
lens: code-tests
outcome: reported
```

**Behaviours and the test that pins each**

1. Sampler Drift rows run only under `drift.enabled` and `drift.scope == full`.
   - Pinned by `TestDriftRows_EntryGate` (`internal/loomrecipe/drift_test.go`).
   - It loops enabled × scope `""`, `"off"` and `"full"` for the sampler and expects the gate to hold only for enabled and full.
   - It also checks tapestry and darn ignore the scope, which pins the "darn and tapestry still behave as before" claim.
2. Sampler row order and `on_done` chain: Record-Write, the three Drift rows, Sampler-Session, Drift-Verify, Code-Bouncer, Drift-Recheck, Describe.
   - Pinned by `TestSamplerRecipe` (`internal/loomrecipe/samplerrecipe_test.go`), whose `want` list now holds the new order.
   - Not confirmed from the diff: whether that test walks `on_done`, or whether it also checks the Drift rows' config (`subject: decision-record`, the `gates` attempts).
3. `fixRoute` routes to Sampler-Session when the run's rows hold it.
   - Pinned by the `AnUnexpectedCellUnderSamplersRows` row in `TestDriftVerify` (`internal/driftengine/verify_test.go`), with `absent: Darn-Write, Webster`.
   - Tapestry's Webster route and darn's Darn-Write route keep their existing rows.
4. The expect agent is told the plan directory only when the recipe has a `Plan-Write` row (`internal/loomcli/wiring.go`).
   - Pinned by none.
   - Grepping `internal/loomcli/*_test.go` for `driftPlanDir` and `PlanDir` finds nothing that asserts the value handed to `driftDeps` for any recipe.
   - `recipekeyed_test.go:61` covers `fixerPlanDir`, a different function.
   - A regression to the old `!darnRun` condition would give a sampler run a plan directory without failing any test.
5. `Sampler.Stamp` with the Drift rows in the recipe.
   - `TestWire_SamplerRecipe` (`wiring_darn_test.go`) now seeds a skipped `Drift-Expect` history entry.
   - It pins the stamp after Drift-Expect.
   - No stamp code changed; the edit only keeps the test valid for the new recipe.
6. Drift verb refusals and help text name sampler runs.
   - Prose only.
   - No test asserts the old strings, so nothing breaks and nothing pins the new ones. This is exempt.
7. `template.yaml` comments and the `doc.go` files change comments only, so they are exempt.

**Findings**

- **MEDIUM, design (test economy):** `internal/loomcli/wiring.go` (the `hasPlan` branch, about line 1035) has no test.
  - Behaviour: the sampler's expect agent gets an empty plan directory, and a recipe with `Plan-Write` still gets one.
  - Evidence: no loomcli test reads the `driftPlanDir` passed into `driftDeps`. See the grep above, and `recipekeyed_test.go:61-66`, which covers only `fixerPlanDir`.
  - Fix: add a row, or a small table, that builds the wiring for the sampler, darn and tapestry recipes and asserts the Drift deps' plan directory is empty for the first two and `planparser.PlanDir(anchor)` for tapestry.
  - Place it beside the existing `TestWire_*` tests in `wiring_darn_test.go`.

I found no duplicated tests. The extended `TestDriftRows_EntryGate` replaces the old tapestry and darn cases rather than repeating them.

**Tool calls**
1. Grep over loomcli and driftengine tests for `driftPlanDir`/`PlanDir`/`DriftPlanDir`, and for the old refusal strings. The manifest held no test-side view of the plan-directory wiring.
