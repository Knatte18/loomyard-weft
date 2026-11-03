---
round: 2
ledger:
  - key: card5-every-engine-build-test-unlisted
    rounds: [1]
    status: resolved
  - key: recipe-spec-stale-for-block-rows
    rounds: [1]
    status: resolved
  - key: arras-motif-interrupt-policy-missing
    rounds: [1]
    status: resolved
  - key: card8-unreachable-profile-recipe-fallback
    rounds: [1]
    status: resolved
  - key: docs-list-recipes-without-arras
    rounds: [1]
    status: resolved
  - key: block-recipes-test-repeats-coverage
    rounds: [1]
    status: resolved
  - key: card7-budget-pass-repeats-card6
    rounds: [1]
    status: resolved
  - key: card8-intent-names-wrong-refusal
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings; round 2 did not raise any of them again, and its review judged card 8's keep-the-name branch justified, so all seven are resolved.
Round 2 raised one new finding, F1 (`card8-intent-names-wrong-refusal`): card 8's Intent names `unsupportedChildRecipeRefusal` where an unembedded recipe name meets `ErrUnknownRecipe` first.
It is a local wording fix the finding text fixes.
