```yaml
lens: fix-consistency
outcome: reported
```

No findings.

I checked three things:
- **Spec vocabulary line:** `contracts/specs/shed-recipe-spec.md:74` lists `clean`, `discussion`, `plan`, `parts`, `description` and `verify`. That matches the order in the error text at `internal/shedrecipe/entries_gate.go:232` and the `case` arms there. No other contract or spec file restates the vocabulary. The only other hit is an unrelated lens-fan list in `docs/research/burler-fork-review-audit-2026-10-16/report.md:60`.
- **New `clean` spec sentence:** it agrees with the resolver, which requires an absolute `WorktreeRoot` (`entries_gate.go:144-145`) and is documented the same way in the doc comment at line 29.
- **New tests:**
  - The new `clean` test sits beside the verify and discussion refusal tests and follows their naming and the `tiergate.Require(t, 0)` convention.
  - The tapestry row's gates (`clean` then `verify`) match `contracts/recipes/tapestry-recipe.yaml:556-558`, which the review quoted.
  - `loomshed.NameCodeBurlerFix` is already used in the same file for the darn and sampler rows.

Calls made:
1. `grep` for the gate-vocabulary wording across `contracts/`, `docs/` and `internal/shedrecipe/*.go`, and for `parts` in `entries_gate.go`. My context lacked any search for other copies of the vocabulary, or for what `parts` requires.
