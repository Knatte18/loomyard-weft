```yaml
lens: fix-affected-set
outcome: reported
```

no findings

F1's set is the golden rows whose Recipe cell can carry a profile. The only fixture is `readmeFixture`, used once, in `TestRenderReadmeGolden`. The T2 row now pins `tapestry · a\|b`, which covers the empty recipe with a profile and the pipe escape. T3 still pins `darn · docs`.

F2's set is the places that describe the Recipe cell. A search for `loom for an empty recipe`, `Recipe column`, `defaultRecipe` and `recipeProfileSeparator` found no stale `loom` wording left. `doc.go:205` already describes the profile suffix, and the render.go constants are consistent with the test comment.

Tool calls:
1. `grep` across internal, docs and README for the old wording and the recipe-cell identifiers. This checked the whole set of sites, which my context lacked beyond `render_test.go`.
