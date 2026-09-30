# Card 10 — shedrecipe-review-entries

**Edit:**
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/registry.go`
- `internal/shedrecipe/entries_simple.go`
- `internal/shedrecipe/entries_simple_test.go`
- `internal/shedrecipe/coverage_guard_test.go`
- `internal/shedbuild/fixture_test.go`

**Create:**
- `internal/shedrecipe/entries_prrework.go`
- `internal/shedrecipe/entries_prrework_test.go`

**Uses:**
- `internal/landingshed/prgate.go`
- `internal/loomshed/prrework.go`

**Intent:**
Register the two new engines in the recipe registry (Shed Recipe Registry Invariant).
`entries_simple.go` gains `prGateEntry`, `publishEntry`'s twin over `landingshed.NewPRGate(env.Landing)`: empty `Config` only, the constructor's own refusals wrapped with the `shedrecipe: PRGate:` prefix, `name` discarded because the gate's identity is its package constant.
`entries_prrework.go` adds `prReworkEntry`, shaped like `planWriteEntry`:
it resolves `gate`/`gate_attempts` through `resolveGateSpec` and rejects any other key, requires `Env.ReworkSpec`, `Env.Shuttle` and every seam of `Env.Rework` through `requireSeam`, and `Env.Rework.PlanDir`/`Env.Rework.ReworkDir` through `requireAbsRoot`;
it builds `shedadapters.NewSingleLLMProducerGated(name, env.ReworkSpec, env.Shuttle, env.Now, nil, gate)` and returns `loomshed.NewPRRework(name, inner, env.Rework)`.
No fresh-spawn preparation is passed: the adapter already archives a stale coverage file on a fresh spawn, and nothing else may be moved out from under the plan.
`recipe.go`'s `Env` gains `ReworkSpec shedadapters.SpecSource` and `Rework loomshed.PRReworkDeps`, a whole-struct passthrough following `Env.Landing`'s precedent, each with a doc comment.
`registry.go` maps `"PRGate"` and `"PRRework"`.
Until the recipe card lands, no consumer reaches either engine, so `coverage_guard_test.go`'s allowlist carries both, each with a comment naming the recipe card that removes it.
Tests: `prGateEntry` rejects unknown config and a nil `TaskHead`, and builds a `*landingshed.PRGate`;
`prReworkEntry` builds with a full `Env`, fails on each missing seam and on an unknown config key, and fails on `gate_attempts` without `gate`.
`internal/shedbuild`'s `TestBuild_EveryRegisteredEngineBuilds` builds every registered name from that package's `newTestEnv`, so in its `fixture_test.go`, `testLandingDeps` fills `ApprovalPath` and `RejectionPath` with paths under its temp directory and `TaskHead` with a stub closure, and `newTestEnv` fills `ReworkSpec` and every `Rework` seam with fakes and absolute temp-dir paths.

**ImpactSummary:** Adds two registry keys and two Env fields; existing entries and callers are unchanged.
