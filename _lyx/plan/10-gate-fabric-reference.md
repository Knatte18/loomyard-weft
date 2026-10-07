# Card 10 — Run card-fabric-reference at both plan gates and validate-plan

**Edit:**
- `internal/planglyph/index.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/validate.go`
- `internal/webstercli/wiring.go`
- `internal/loomcli/parity_test.go`
- `internal/loomcli/validate_test.go`
- `internal/loomshed/gates_test.go`
- `internal/loomshed/gatefindings_test.go`
- `internal/planglyph/delta_integration_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/websterengine/gitfake_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`

**Intent:**
Put card 9's check into the plan-format validation both plan gate sites and `lyx loom validate-plan` already share (`PATTERN-gate-self-check-parity`), and steer plan writers to committed data (D13, D15).

`planglyph.NewIndex` takes the matcher card 9's seam declares and holds it; the index's `ValidateFormat` and `ValidateRework` append the check's converted findings, as blocking, to what they return today.
`ValidateDispatch` does not run it: D13 places the check at Plan-Gate, and webster's dispatch re-resolution is out of scope.
Callers pass card 8's location-free rule: `internal/loomcli/wiring.go` (both `NewIndex` sites) and `internal/webstercli/wiring.go` (both, where the index serves only dispatch); every test site passes the same real rule, so no test double of the rule exists.
The Plan-Write and Plan-Burler gates (`loomshed.NewPlanGate` over the index) and `validate-plan`'s default and `--rework` modes then report the finding; `validate-plan`'s Long help says its default mode includes it.
Wording outside the fabric side says "fabric repo" (`PATTERN-fabric-vocabulary`).

`loom-plan-spec.md`: add the `card-fabric-reference` row to the validation checks list, naming what it scans, the two rules and their spans, the finding's way forward, that it runs in the index's format validation at both gate sites and `validate-plan` but not at dispatch, and its bound (what it misses; the implementer audit stays the guard), in the commit that changes the gates (`PATTERN-refusal-way-forward`).
`loom-template-plan.md`: tell the writer that a card needing a corpus or fixture takes it from committed test data in the tree, created by an earlier card where it does not exist yet, never from live files of other worktrees or the fabric repo, and point at `card-fabric-reference` by ID for what is refused without restating its rule (`PATTERN-producer-pointer-rule`).

Tests:
- `parity_test.go`: a row in `TestGateParity_PlanGate` whose plan carries a sibling path in a card Verify gets the same `card-fabric-reference` finding from the gate closure and from `validate-plan`.
- `gates_test.go`: a `TestNewPlanGate` row where that plan fails the gate with the finding, and a `TestNewReworkPlanGate` row where the rework gate reports it too.
- `validate_test.go`: a `TestValidatePlanCmd` row where the envelope carries the finding.
Build paths with `fabricengine.WeftWorktree` over a synthetic location and spellings from `lyx fabric …`, as card 9's tests do.

**ImpactSummary:** the plan index constructor gains one parameter that every construction site supplies; the format and rework validations gain one check; dispatch validation is unchanged.
