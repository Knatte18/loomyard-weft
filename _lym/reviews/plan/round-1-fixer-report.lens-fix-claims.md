```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every sentence the fix added or rewrote that states a fact about the code at HEAD.

**MEDIUM — card 18, the `validate-plan --part` recipe check: "takes no reaching test, as `refusal-spec.md` allows for a refusal no on-disk state a run can produce reaches"**
- **Summary:** The fix claims a reaching-test exemption that `refusal-spec.md` grants only to a fail-loud wiring guard. This refusal is an operator-facing message with a way forward, not a fail-loud guard.
- **Evidence:** `contracts/specs/refusal-spec.md:32` defines the wiring guard class as "nil deps, an invalid producer list, empty paths; unreachable from any on-disk state a run can produce | fail loud; no reaching test".
- **Evidence:** `contracts/specs/refusal-spec.md:42` exempts only "a fail-loud wiring guard, which no on-disk state a run can produce reaches."
- **Evidence:** The card's own text for this refusal ends in "way forward: run \"lym loom validate-plan\" without --part", so it is not fail loud.
- **What does hold:** The unreachability itself is true. `contracts/recipes/tapestry-recipe.yaml` is the only recipe with `engine: PlanWrite`, and `loomrecipe.HasEngine` parses embedded recipes only (`internal/loomrecipe/recipekeys.go:45`).

**LOW — card 12 ImpactSummary: "five pointers to the format name them"**
- **Summary:** The count is wrong.
- **Evidence:** The card's own **Pointers to the format** paragraph lists six spec and doc pointers: three in `loom-plan-spec.md`, two in `webster-spec.md` and one in `docs/overview.md`. The rubric pointer makes seven.

**Claims I checked that hold:**
- **Comment stripping:** an included block's leading comment is stripped when it renders (`internal/stencil/stencil.go:111` parses `stripLeadingComment(body)`).
- **Trailing newline:** `seat-directive-chair.md` ends without a trailing newline.
- **Marker rows:** `TestStencils_TopLevelMarkers` accepts a `nil` marker list, as `contracts/stencils/markers_test.go:29` shows.
- **Prompts:** `seatengine.Prompts` exists (`internal/seatengine/prompt.go:96`).
- **Line ranges:** the block boundaries 54–68, 78–110, 113–165 and 167–279 match the stencil, and so do each block's markers.
- **`Env.Setting`:** its signature is `func(module, key string) (any, error)` on `Env`.
- **The `parts` gate row in tests:** `envkit.FullEnv` fills `Setting` with `TemplateSetting`, so its `parts` row reads `parts_max` from the template once card 1 adds it.
- **Quoted doc sentences:** the quotes in cards 5, 7, 9 and 17 match the doc lines at HEAD, including the mode-list sentence at `contracts/specs/loom-plan-spec.md:414`.
- **Canonical spelling:** the claim that canonicalization keeps the unit fits `CanonicalizeHandles`. It builds the declaration's unit from `HandleUnit` of the draft and rejects any Name result whose unit differs.

Tool calls:
1. Grepped `internal/stencil/stencil.go` and seatengine to see whether includes have their comment stripped and whether `Prompts` exists; my context had only the `stripLeadingComment` definition, not its call sites.
2. Read `contracts/stencils/markers_test.go` and grepped the recipes, to check that a `nil` marker row is accepted and which recipes hold `PlanWrite`; my context had neither.
3. Grepped envkit's `Setting` fill, to check the `parts` gate row once `PartsMax` was removed from `FullEnv`; my context lacked it.
