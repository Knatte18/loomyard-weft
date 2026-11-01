```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 and F2 proposals as carried out in `internal/boardengine/render_test.go`.

F1 (give T2 `Profile: "a|b"`, golden cell `tapestry · a\|b`):
- `recipeCell` (`render.go:746-755`) returns `defaultRecipe + recipeProfileSeparator + cellText(t.Profile)` for an empty recipe with a profile. The golden pins the default-recipe branch and the pipe escaping together. Dropping either now fails the test.
- T2 is the only task whose row changes. The fixture is used only by this one golden row (the grep showed a single `readmeFixture()` call at line 298), so no sibling golden breaks.
- The Go string `a\\|b` evaluates to `a\|b`, which matches the cell the doc comment says is escaped.
- T2 carries `Status: "running"` as a hand-set word. The review said the row does not render a Running subsection for it, and the golden still shows T2 in Layer A, so the added `Profile` does not change its placement.
- A profile of `a|b` is not a realistic validated run-profile name, which is a fixture-realism point only. The review itself proposed `a|b`, and the rubric excludes extra improvements.

F2 (reword the doc comment):
- The new sentence says `tapestry` is the empty-recipe value, matching `defaultRecipe` and the golden rows.
- It also says the profile is appended after ` · `. I did not verify that `recipeProfileSeparator` is exactly that string. The golden row `darn · docs` (line 333) shows the separator is space, middle dot, space, so the comment agrees.
- The added fixture-doc clause "a profile with a pipe on a task with no recipe, a recipe with a profile" matches T2 and T3 exactly.

No findings.

Tool calls made: none. My context already held every file I needed in its changed form.
