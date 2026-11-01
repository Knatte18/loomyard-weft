---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/boardengine/render_test.go:252
    origin: lens:code-tests
    summary: The empty-recipe-with-profile case (tapestry · profile) and pipe escaping of the profile have no golden row.
    fix: In render_test.go give a fixture task with an empty recipe (for example T2 top, or the Draft task) a Profile containing a pipe such as a|b, and change its golden row's Recipe cell to tapestry · a\|b. This extends the existing TestRenderReadmeGolden rows; add no new test.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/boardengine/render_test.go:267
    origin: lens:code-affected-set
    summary: The TestRenderReadmeGolden doc comment says loom for an empty recipe and omits the profile that the Recipe cell now carries.
    fix: Reword the sentence at render_test.go:267 to say each Tasks table pins a Recipe column after the slug, tapestry for an empty recipe and the stored name otherwise, followed by ` · ` and the profile when the task names one, and the Notes and Done tables pin its absence.
---

### [LOW] Profile-bearing Recipe cells are pinned only for a non-empty recipe

**Location:** internal/boardengine/render_test.go:252
**Issue:** The diff adds two behaviors to `recipeCell`: an empty recipe with a profile renders `tapestry · <profile>`, and the profile goes through `cellText`.
The only fixture with a profile is T3 (recipe `darn`, profile `docs`), so dropping the `defaultRecipe` branch when a profile is set, or dropping `cellText` around the profile, passes every test.
The fix extends an existing golden row rather than adding a test.

### [LOW] Stale golden-test doc comment

**Location:** internal/boardengine/render_test.go:267
**Issue:** The comment says the Recipe column shows `loom` for an empty recipe, but `defaultRecipe` is `tapestry` and the rows show `tapestry`.
The diff changes the cell's contract and the golden row at line 333, yet the comment does not mention the profile suffix.

## Rejected

- Whitespace-only profile yields `tapestry · ` with a trailing separator (code-correctness lens, noted as no defect): the profile name comes from a validated run-profile name, not untrusted input.
- Done tasks no longer show their profile anywhere (code-correctness lens, scope observation): the Idea addresses the Recipe column only, and the Done table has no Recipe column by the existing contract.
- Separate NIT for the stale test comment (code-tests lens): merged into F2.
