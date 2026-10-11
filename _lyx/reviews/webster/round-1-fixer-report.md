# Webster round 1 — fixer report

## Changes

- **F1** (NIT, consistency), fixed as proposed in `d6eb9877a`.
  The premise holds: `renumberWaysForward`'s slug step read "pass a slug of lowercase letters and digits joined by single hyphens", while the `insert-card: invalid slug` row in `contracts/specs/refusal-spec.md` and card 4's way forward read "pass lowercase letters and digits joined by single hyphens".
  The code string in `internal/loomcli/renumber.go` now matches the spec row and the plan, and the `insert: invalid slug` row of `TestRenumberVerbs_Refusals` asserts the new wording.
  `lyx gate test ./internal/loomcli` passes.

## Disputed

None.

## Deferred

None.
