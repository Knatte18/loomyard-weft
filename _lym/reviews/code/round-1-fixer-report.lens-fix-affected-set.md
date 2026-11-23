```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the F1 fix (the CONVERGED rule in `harden-rubric-review.md`) for other sites that still tie convergence to a reviewer-model change.

No findings.

- **Stale clause:** no other text in `contracts/stencils/harden`, `contracts/recipes/hardener-recipe.yaml` or the non-test files of `internal/hardenshed` and `internal/hardenengine` still requires a changed reviewer model for convergence.
- **Edited rule:** the only CONVERGED rule is `harden-rubric-review.md:98`. It now reads "no BLOCKING or MEDIUM finding and whose live gate is green", and line 99 says "both", so no "all three" or three-condition wording is left.
- **Other rotation text:** `internal/hardenshed/doc.go:13-14` and `:97` and `template.yaml:5-6` already state the one-model default with rotation as a per-run choice. `campaign.go:48` and `:120-130` refuse neighbouring repeats only in lists of two or more. None of these conflicts with the fix.

Calls:
1. `git grep` for rotation and model-change wording across the harden stencils, recipe and hardenshed/hardenengine non-test code, to find other sites with the stale premise. Context only held the one stencil span.
2. `git grep` for "all three", "three conditions" and CONVERGED in the same paths, to find leftover wording around the edited rule.
3. A broader `git grep` for rotation wording across `contracts`, `internal/hardenshed` and `docs`. It was too noisy to use, and call 1 replaced it.
