# Card 7 — rework-gate-whole-plan

**Edit:**
- `internal/loomshed/gates.go`
- `internal/loomshed/gates_test.go`
- `internal/loomcli/validate.go`
- `internal/loomcli/validate_test.go`

**Prosa:**
- `docs/overview.md`
- `CONSTRAINTS.md`

**Intent:**
Give the `rework-plan` gate its new meaning (D5): the whole live plan is held to the plan gate's format-only check set, plus one rework check that `first_card` equals the number Go told the session.

`ValidateReworkPlan` keeps its signature, so `NewReworkPlanGate` and `lyx loom validate-plan --rework` keep calling the one shared function (Gate Self-Check Parity Invariant).
It runs `planglyph.ValidateFormat` over the whole plan in place of `ValidateDispatch` scoped by the committed cards, and appends a blocking finding (check `rework-first-card`) when `plan.FirstCard` differs from `NextReworkCardNumber` over the plan committed at HEAD.
That told number is stable for the gate's whole window: HEAD still holds the retired generation from the archive until the round commit, and the gate runs only inside that window.
A plan that cannot be read at HEAD stays a returned error, never a finding.

`validate-plan`'s `--rework` long help and flag text, the `NewReworkPlanGate` and `ValidateReworkPlan` doc comments, `docs/overview.md`'s `validate-plan --rework` line and the PR-Rework entry of `CONSTRAINTS.md`'s Gate Self-Check Parity line describe the new check: the whole new plan, plus the told `first_card`.

Tests in `gates_test.go`: a whole new plan numbered from the told card passes; a `first_card` differing from the told number fails with `rework-first-card`; a glyph in any card that does not resolve against the current tree is a blocking finding (no card is exempted as built); an unreadable committed plan is a returned error.
`validate_test.go` asserts the `--rework` verb reports the same findings as the gate for the mismatched `first_card` case.

**ImpactSummary:** one shared validator changes meaning; its two callers (PR-Rework's gate closure and `validate-plan --rework`) change with it and nothing else calls it.
