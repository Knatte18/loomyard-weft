```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the changed code and test through the correctness lens.

No findings.

- **`recipeCell`, T2 case.** T2 has `Recipe: ""` and `Profile: "a|b"`. The recipe branch is skipped, so `recipe` stays `defaultRecipe`, which is `tapestry`. The profile is non-empty, so the function returns `tapestry` + `recipeProfileSeparator` + `cellText("a|b")`. `cellText` escapes the pipe, as the existing `Top \| work` row shows. The result is `tapestry · a\|b`, which matches the new golden cell `tapestry · a\\|b` once Go string escaping is read. The gate run was green: 559 passed, 0 failed.
- **Defect check.** With the fix in place, three mutations of `recipeCell` would each fail the new row:
  - Dropping the default recipe when a profile is set would give `a\|b` or ` · a\|b`.
  - Dropping `cellText` around the profile would give `a|b` unescaped.
  - Dropping the profile suffix would give `tapestry`.
  
  Before this fix, T3's `darn · docs` row passed under the first two mutations.
- **Callers.** The change is test-only, with no signature or behaviour change in production code. `readmeFixture()` has a single caller, `TestRenderReadmeGolden` at `render_test.go:298`, so no other golden is affected by T2's new `Profile`.
- **Fixture description and doc comment.** The edited fixture comment and the `TestRenderReadmeGolden` Recipe sentence match the code. The default is `tapestry`, and the profile suffix is pipe-escaped.

Tool calls: none beyond the ones already in the context.
