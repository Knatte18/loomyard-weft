# Round 2 fixer report — plan

## Changes

- **F1** (fixed as proposed): card 6's Intent, after the `PATTERN.md` entry line, now says `PATTERN.md` ends at no more than 15564 bytes (`pattern.HeadroomBytes`), so the card also shortens existing entry lines by at least 264 bytes in total, keeping the declared line verbatim and dropping only words the linked background files already say, with `lyx loom lint-pattern` confirming.
  Checked: `PATTERN.md` is 15560 bytes, `HeadroomBytes = MaxOverviewBytes * 95 / 100` is 15564, and the declared line is 268 bytes with its newline.
- **F2** (fixed as proposed): card 1's test paragraph splits the assertion, so the unregistered-module row asserts the module and `way forward`, and the unknown-key row asserts the module, the key and `way forward`.
- **F3** (fixed differently): card 5's `loomSegmentPerches` comment now names it the segment-to-perch table the segment settings are read through, and says a new loom perch also adds a row to `loomPerches`, which the hard-cap lookup keeps.
  The proposed text named `loomSegmentBudgets` as a reader, but card 7 creates that function (`07-budgets-by-segment.md:6`), after card 5, so at card 5 the comment would cite a function that does not exist yet.
- **F4** (fixed as proposed): card 3's `**Edit:**` gains `contracts/specs/shed-recipe-spec.md`, and its Intent rewords the spec's `Env`-versus-`Config` rule to the new rule.
  The rule keeps the original's closing clause, "resolved against one of those roots by the entry that reads it" (self-check, fix-proposal).
  The Intent also drops `only` from the `ReviewModels` comment's "Env carries roots and run-wide values only" (`internal/shedrecipe/recipe.go:119`), which card 8 deletes later (self-check, fix-affected-set and fix-proposal).
  Card 3's `**ImpactSummary:**` now names the spec rewording (self-check, fix-consistency).
- **F5** (fixed as proposed): card 1 now reads `configengine.ConfigFile(<board dir>, "loom")`.

`lyx loom validate-plan` reports `ok` after every change.

## Self-check

- fix-claims: no findings.
- fix-consistency, LOW: card 3's `**ImpactSummary:**` did not mention the spec edit. Fixed as proposed.
- fix-affected-set, LOW: `internal/shedrecipe/recipe.go:119` still stated the replaced "only" rule. Fixed as proposed, with its fix-proposal duplicate merged in.
- fix-proposal, LOW: the reworded spec rule dropped the clause that says how a `Config` path is resolved. Fixed as proposed.
- fix-proposal, LOW: the stale `recipe.go:119` twin. Duplicate of the fix-affected-set finding; fixed there.

## Disputed

None.

## Deferred

None.
