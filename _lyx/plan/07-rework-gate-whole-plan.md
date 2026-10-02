# Card 7 — rework-gate-whole-plan

**Create:**
- `internal/planparser/firstcard.go`
- `internal/planparser/firstcard_test.go`

**Edit:**
- `internal/planglyph/planglyph.go`
- `internal/planglyph/planglyph_test.go`
- `internal/loomshed/gates.go`
- `internal/loomshed/gates_test.go`
- `internal/loomcli/validate.go`
- `internal/loomcli/validate_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`
- `docs/overview.md`
- `CONSTRAINTS.md`

**Intent:**
Give the `rework-plan` gate its new meaning (D5): the whole live plan is held to the plan gate's format-only check set, plus one rework check that `first_card` equals the number Go told the session.

The `first_card` comparison lives in `internal/planparser`, per the decision record's Planparser Sole-Parser constraint (`first_card` is parsed and checked only there):

- `firstcard.go` adds `CheckFirstCard(plan *Plan, told int) []ValidationError`, returning one finding with check `rework-first-card`, naming both numbers, when `plan.FirstCard` differs from `told`, and none otherwise.
  It is outside both `ValidateFormat` and `Validate`, so the twenty-eight-ID set and every count that names it stay unchanged; its doc comment says the rework gate alone runs it.
- `planglyph` gains `ValidateRework(plan *planparser.Plan, worktreeRoot string, told int) ([]Finding, error)`: `ValidateFormat`'s findings, then `CheckFirstCard`'s converted blocking through the package's own converter, with `ValidateFormat`'s error contract.

`ValidateReworkPlan` keeps its signature, so `NewReworkPlanGate` and `lyx loom validate-plan --rework` keep calling the one shared function (Gate Self-Check Parity Invariant).
It computes the told number with `NextReworkCardNumber` over the plan committed at HEAD and returns `planglyph.ValidateRework` over the whole plan, in place of `ValidateDispatch` scoped by the committed cards; it reads `plan.FirstCard` nowhere itself.
That told number is stable for the gate's whole window: HEAD still holds the retired generation from the archive until the round commit, and the gate runs only inside that window.
A plan that cannot be read at HEAD stays a returned error, never a finding.

`loom-plan-spec.md` documents `rework-first-card` in a paragraph after the "Validation checks" list: a rework-gate-only check outside both entry points, which `planglyph.ValidateRework` runs after the format-only set.
`validate-plan`'s `--rework` long help and flag text, the `NewReworkPlanGate` and `ValidateReworkPlan` doc comments, `docs/overview.md`'s `validate-plan --rework` line and the PR-Rework entry of `CONSTRAINTS.md`'s Gate Self-Check Parity line describe the new check: the whole new plan, plus the told `first_card`.

Tests: `firstcard_test.go` covers an equal `first_card` (no finding), a differing one (one `rework-first-card` finding naming both numbers) and an absent one checked against a told `1` (no finding).
`planglyph_test.go` asserts `ValidateRework` returns `ValidateFormat`'s findings followed by a blocking `rework-first-card` for a mismatch, and exactly `ValidateFormat`'s findings for a match.
`gates_test.go`: a whole new plan numbered from the told card passes; a `first_card` differing from the told number fails with `rework-first-card`; a glyph in any card that does not resolve against the current tree is a blocking finding (no card is exempted as built); an unreadable committed plan is a returned error.
`validate_test.go` asserts the `--rework` verb reports the same findings as the gate for the mismatched `first_card` case.

**ImpactSummary:** one shared validator changes meaning; its two callers (PR-Rework's gate closure and `validate-plan --rework`) change with it and nothing else calls it; the new planparser check and planglyph entry point are additive and run only on the rework gate.
