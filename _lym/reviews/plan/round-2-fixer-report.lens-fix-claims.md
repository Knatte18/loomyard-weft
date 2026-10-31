```yaml
lens: fix-claims
outcome: reported
```

I checked the claims the fix added or rewrote against the code at HEAD. One wording problem; every other claim holds.

**MEDIUM**: Card 3's Intent overstates what the citation scan refuses.
- **Changed text:** `_lym/plan/03-sampler-stencil.md`, Intent: "the stencil carries no `docs/`, `contracts/` or `internal/` path, because `TestStencils_NoBareCrossRepoCitations` refuses one".
- **Summary:** the scan does not refuse every path under those prefixes. It refuses only a token that ends in `.md` or `.go`, has no `#`, and is not `plan:`-prefixed.
- **Evidence:** the token rule in `contracts/stencils/citation_enforcement_test.go`:7-9 and `citationIsBare` at :75-82. The file's own comment notes that a bare package reference such as `internal/planglyph` is deliberately left out of scope.
- **Possible rewording:** "carries no bare `docs/`, `contracts/` or `internal/` file citation, which `TestStencils_NoBareCrossRepoCitations` refuses".
- The rest of that sentence holds: `docs/sandbox-howto.md` exists, and as a bare `.md` token it would be refused.

Claims that hold:
- **Card 5:** `"SamplerSession"` sorts between `"RecordWrite"` and `"SeedChild"` in the `ShipsExpectedEntries` list (registry_test.go:85-114). `checkLoomContract` holds the engine-choice text "Implementer, DarnWrite or BeamAwait" (shedbuild/loomcontract.go:25). Outside tests, `loomrecipe.verifyGatedRows` is the only other `LoomBuildEngines` consumer (darnrecipe.go:34).
- **Card 8:**
  - Production callers: `codeBudgetZero` has two, `landingDeps` (landingdeps.go:272) and `rerouteReviewed` (:393). `rerouteReviewed` has one, `landingDeps` (:82).
  - One test calls both, in landingdeps_test.go:531-546.
  - `wireDarn` currently takes `budgets map[string]int` (darnwiring.go:198), so dropping it is a real change.
- **Card 7:** `childSeedParams` returns `params.parent` for any recipe `childRecipeSupported` admits (battencli/wire.go:549-560). The shared bootstrap writes `params.parent` through `loomSeedFor(seedRecipe, parent, driver)` (sharedbootstrap.go:112). The `HandBack` doc reads "a tapestry or darn escalate halt" (battenshed/deps.go:277).
- **Card 11:** `loomcli/doc.go` holds the decision-add caller-rule sentences (doc.go:36 and :284).
- **Card 2:** the corrected "a test row for each" matches its Intent, which already says "gains a row for each".

Tool calls:
1. One bash call that printed `verifyGatedRows` and `checkLoomContract`, the callers of `rerouteReviewed` and `codeBudgetZero` (production code and tests), the `wireDarn` signature, and the decision-add lines in `loomcli/doc.go`. My context held none of that code.
