# Card 7 — plan-gate-done-card-edited

**Edit:**
- `internal/loomshed#ValidatePlan`
- `internal/loomshed/gates_test.go`
- `internal/loomshed/doc.go`
- `internal/loomcli/parity_test.go`
- `//contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/websterengine#EditedDoneCards`

**Intent:** Refuse an edit to a done card at the plan gate instead of at the Webster row (D2).
`ValidatePlan` keeps telling the index the done cards, and then appends one blocking `done-card-edited` finding per id `EditedDoneCards` returns, naming the card and the follow-up landing: a done card is frozen, and a finding against it lands as a follow-up card after the last begun batch with its Card Index line in `00-overview.md`.
The check reads webster's record and the card bytes; with no record or no recorded hashes it reports nothing, and an unreadable card file is a returned error like an unreadable record.
It gates the done-card edit only; a begun batch's card-set change, a failed batch with uncheckable findings and an overview change outside the Card Index pass the gate and block at the Webster row.
The gate closure and `lyx loom validate-plan` share `ValidatePlan`, so PATTERN-gate-self-check-parity holds without a second site.
`TestNewPlanGate` gains the acceptance rows: an edited done card fails with `done-card-edited`, and an unedited done card, an unbegun card, a plan with no record and a record with no recorded hashes each report nothing from the check; a done card whose file cannot be read is a returned error from the gate.
`TestGateParity_PlanGate` in `parity_test.go` gains a `DoneCardEdited` fixture, a plan with a webster record whose done batch's recorded hash differs from its card file, on which the gate and `validate-plan` both report `done-card-edited`.
`loomshed`'s `doc.go` names the finding beside the done-card scoping, and the plan spec's validation-checks section gains a row for `done-card-edited` as a check `loomshed.ValidatePlan` appends outside both planparser entry points, the way `card-fabric-reference` is documented.

**ImpactSummary:** One loomshed function and its test; both plan gate sites and the validate verb pick the finding up through it.
