# Card 4 — loomshed-index-seam

**Edit:**
- `internal/loomshed/gates.go`
- `internal/loomshed/doc.go`
- `internal/loomshed/gates_test.go`
- `internal/loomshed/gatefindings_test.go`
- `internal/loomshed/seam_enforcement_test.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_gate.go`
- `internal/shedrecipe/entries_gate_test.go`
- `internal/testkit/envkit/envkit.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/validate.go`
- `internal/loomcli/parity_test.go`
- `internal/loomcli/validate_test.go`

**Uses:**
- `plan:internal/planindex#Index`
- `plan:internal/planglyph#NewIndex`

**Intent:**
Decision D8's seam for `loomshed`: its production code stops importing `internal/planglyph`, so `shedrecipe`, `shedbuild`, `loomrecipe`, `battenrecipe`, `battencli`, `envkit` and the other packages that reach `loomshed` stop linking tree-sitter.

`NewPlanGate`, `NewReworkPlanGate` and `ValidateReworkPlan` each take a `planindex.Index` parameter and call its `ValidateFormat` or `ValidateRework` instead of `planglyph`'s.
Finding types are named through `planindex`.
`seam_enforcement_test.go`'s import allowlist swaps `internal/planglyph` for `internal/planindex`, so the guard keeps `loomshed` off the index for good.

`shedrecipe.Env` gains `PlanIndex planindex.Index`.
The "plan" and "rework-plan" gates require it non-nil, through the same `requireSeam` check the "rework-plan" gate already applies to `Rework.ReadCommitted`, so a row naming either gate without the wiring fails to build with the key's name rather than at run time.
`loomcli/wiring.go` fills it with `planglyph.NewIndex()`.
`validate-plan` calls the same `Index` methods its gates call, so `PATTERN-gate-self-check-parity` holds by construction, and `parity_test.go` passes `planglyph.NewIndex()` to the gate constructors it compares.

`envkit` fills `PlanIndex` with a stub that answers no findings, so the packages built on it do not link the index.
A test that asserts real resolve-backed findings, as `loomshed`'s own gate tests do, passes `planglyph.NewIndex()` itself, and its package's test binary stays on card 5's allowed list.
`entries_gate_test.go` gains the row that a "plan" gate with a nil `PlanIndex` fails to build, naming `PlanIndex`.

**ImpactSummary:** Three exported `loomshed` constructors gain an `Index` parameter, with callers in `shedrecipe` and `loomcli` only; `Env` gains one field filled by `loomcli` and `envkit`.
