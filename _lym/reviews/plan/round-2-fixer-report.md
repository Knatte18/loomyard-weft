# Round 2 fixer-report — plan

## Changes

- F1, LOW, consistency — fixed as proposed.
  Checked the premise against the code: `WriteSeed` (`internal/battencli/wire.go:1392`) calls `shedrun.ValidateRecipe` and wraps its refusal in `battenshed.ErrUnknownRecipe` before `childRecipeSupported` and `unsupportedChildRecipeRefusal` are reached (`:1399-1400`). The refusal names the available recipes (`internal/shedrun/seed.go:96`).
  In `_lym/plan/08-profile-recipe.md`, the Intent now says that a task whose recipe names no embedded recipe, with no profile named, passes `prepareRunSettings` with its settings alone and is then refused by the `WriteSeed` seam through `shedrun.ValidateRecipe`, wrapped in `battenshed.ErrUnknownRecipe`, which names the available recipes. It also says that a task naming a profile is still refused by `profiles.Settings`, because the profile holds no block for that name.
  The `unsupportedChildRecipeRefusal` clause is gone. No target or verify line changed.

## Self-check

- fix-consistency, LOW: `_lym/plan/lens-review.md:15` repeated the wrong `unsupportedChildRecipeRefusal` claim. Fixed as proposed: the clause now names the `shedrun.ValidateRecipe` refusal wrapped in `battenshed.ErrUnknownRecipe`. The verbatim lens fork reports behind that line were left as they are.
- fix-affected-set, LOW: the same `lens-review.md:15` finding. Merged into the fix-consistency finding above.
- fix-claims: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
