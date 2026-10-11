---
round: 1
focus:
  - "Card 10 deletions: confirm every `Delete` group's `assert-no-callers` ran against each of its own targets (the one-row round producer, the two-seat engine round, its stencils, the ready marker, the old row-name constants), and that no reference survives in code, tests, recipes or testdata."
  - "Acceptance record (`round-<n>-acceptance.md`): only the review row writes it, only after `ParseCurrentReview` passes, and every whole-round archive moves it with the review and the fixer report; settle whether the changed-review hard error carries the goto-to-review-row way forward."
  - "Fix row spawn condition: the fixer spawns only for a review with findings or a clean review whose row gates fail; settle whether a clean review with passing gates skips it and whether `lyx burler run` matches the recipes."
  - "Auto-accept settle and per-perch keys: a Bouncer told `AutoAccept` settles a spent budget as an accept; settle whether 0 and absent fall back to the run-wide `review_max_bounces` in one place (the recipe builder), keyed by each review segment's Bouncer row name, with the engine and routing projection applying the same value."
  - "Test economy: new tests in the halves, producers and wiring cards against existing coverage, and whether the diff stays within the per-package rows of `cmd/lyx/testdata/test-budget.yaml`."
  - "Doc landing: module docs in `doc.go`, `docs/overview.md` and `PATTERN.md` or `pattern/` files updated in the same change for the new rows, the removed ready marker and the new `loom.yaml` keys."
---

Seed focus only; every entry names where to look, and the full diff stays in scope.
