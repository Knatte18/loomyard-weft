# Fixer report — plan, round 2

## Fixed

- **F1 (LOW):** premise confirmed at `internal/planparser/validate.go:472`, where `checkGlyphMalformed`'s doc comment says "containment-unit-overlap skips it the same way".
  Card 5's (`05-retire-cross-card-containment.md`) `validate.go` sentence now also states that this doc comment stops listing `containment-unit-overlap` among the checks that skip a malformed glyph.
  The other remaining mentions of the retired tiers in `planglyph.go`, `handle.go`, `repo.go`, `planglyph_test.go`, `status_enforcement_test.go` and `beginbatch.go` are already assigned to card 5's members and files, and `handle_test.go`'s are assigned to card 6.

`lyx loom validate-plan` over the edited plan returns `ok: true` with informational `create-new-unit` findings only.

## Disputed

None.

## Deferred

None.
