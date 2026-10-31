```yaml
lens: fix-affected-set
outcome: reported
```

I reran the searches for the four set-premise findings (F1, F3, F4 and F5); I left none unchecked. F2's set is a single doc line, so I also ran its search.

no findings

- F1: the other tests that list `"RecordWrite"` are `loomrecipe/recipe_test.go`, `darnrecipe_test.go` and `recipekeys_test.go`. They pin rows or engines for each recipe, not the registry's name list. `TestNames/ShipsExpectedEntries` is the only test that pins the full registry name list, and card 5 now targets it.
- F2: `AmendmentTarget`'s doc at `internal/loomrecipe/names.go:84` is the only loomrecipe doc that lists the four recipe constants. Card 6 now targets it.
- F3: in Go, every "tapestry or darn" comment that a sampler run now falls under sits in a symbol or file that card 7 targets. That includes `specFor` (`arm.go:408`), `battenCLI.wire` (`wire.go:1086` and `wire.go:1273`) and `loomArbiterSeam` (`drivewiring.go:129`), plus the two that were added, `childSeedParams` and `InnerRunDeps.HandBack`. The comments left unlisted (driftcli, reportengine, shedengine/doc.go:59, finalizerow, bouncerrows, the burlerengine fan-key line) only describe the tapestry and darn recipes' own files or behaviour, and that stays true. In markdown, the files under `contracts` and `docs` that say "tapestry or darn" or "tapestry and darn" are the three specs, the Code-Review rubric, the orch role stencil and `docs/overview.md`, and card 7 lists all six.
- F4: the stencil text card 3 plans now carries no other `docs/`, `contracts/` or `internal/` `.md` or `.go` token. The `internal/shellguard` mention is in the `SessionSpec` sentence, not in the stencil.
- F5: the call sites in the repository match the corrected card 5 and card 8 summaries. `codeBudgetZero` is called at `landingdeps.go:272` and `landingdeps.go:393`, and `rerouteReviewed` at `landingdeps.go:82`. `LoomBuildEngines` is used by `loomcontract.go:25` and `darnrecipe.go:34`.

Calls made, each because my context did not hold it:
1. Grepped tests for `"RecordWrite",` and `"SeedChild"`: the full set of registry name pins was not in context.
2. Grepped loomrecipe non-test Go for the recipe-constant list: the F2 set was not in context.
3. Grepped non-test Go comments for "tapestry or/and darn": the F3 Go set was not in context.
4. Grepped for the callers of `codeBudgetZero`, `rerouteReviewed`, `darnDeps` and `wireDarn`, and the users of `LoomBuildEngines`: the full F5 call sites were not in context.
5. Used awk to find the function enclosing four comment lines: needed to map those comments onto card 7's targets.
6. Grepped `contracts` and `docs` markdown for "tapestry or/and darn": the F3 markdown set was not in context.
