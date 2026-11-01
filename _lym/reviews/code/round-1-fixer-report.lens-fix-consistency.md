```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set in `render_test.go` for agreement with its surroundings.

no findings

- The `render_test.go:267` doc comment now matches `doc.go:205-206`, which says `tapestry` for an empty recipe, then ` · ` and the profile, and no Recipe column on the Notes and Done tables.
- The `readmeFixture` doc comment lists the new T2 profile and the T3 recipe with profile.
- No other test or doc uses the fixture or still says `loom` for an empty recipe, so no twin was left stale.

Tool calls:
1. `grep` for "recipe column", "loom for an empty", `defaultRecipe` and `recipeProfileSeparator` across `.go` and `.md` files, to find twins of the changed text; my context held only `render.go:746-755` and `render_test.go`.
