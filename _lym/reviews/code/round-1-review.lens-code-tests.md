```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour the diff adds or changes, the test row that pins it.

**Per-behaviour list**

1. Recipe cell appends ` · <profile>` when a profile is set (recipe non-empty): pinned by the golden row `T3 … | darn · docs |` (`render_test.go:333`, fixture `render_test.go:252`).
2. Recipe cell shows the recipe alone when no profile is set: pinned by the existing rows for `tapestry` (e.g. `render_test.go:341`, `:442`) and the `darn` row at `render_test.go:419`.
3. The profile bullet is gone from the entry cell: pinned by the T3 row, `render_test.go:332`. Its title cell holds only the setting bullet, and a bullet left in place would fail the golden row.
4. Empty recipe with a profile, which renders `tapestry · <profile>`: **none**. The only fixture with a profile (T3) has recipe `darn`. If the `defaultRecipe` branch were skipped when a profile is set, for instance by returning `recipe` as an empty string, every test would still pass.
5. Pipe escaping of the profile via `cellText`: **none**. The fixture profile is `docs`. Dropping `cellText` around `t.Profile` would pass. The pipe-escaping rows in the title and setting cells (`render_test.go:325`) do not touch the profile.
6. The profile shown in the Running table's Recipe cell (the `At`/`Owner` table, `render.go:493`): **none**. The Running fixture has no profile (`render_test.go:419` has `Recipe: "darn"` only). The Ready/layer/Draft table and the Running table share `recipeCell`, so one function covers both. A divergence per table would be undetected, but that is only possible if someone stops calling `recipeCell`. This overlaps with item 4 and is the same gap.

**Findings**

- **LOW, design.** `internal/boardengine/render_test.go:252`, `:333`. The diff adds a branch for an empty recipe with a profile (`tapestry · <profile>`) and runs the profile through `cellText`. No fixture row exercises either. Fix: give an existing task with an empty recipe (for example T2 `top` or a Draft task) a profile containing a pipe, such as `a|b`, and update its golden row to `tapestry · a\|b`. That covers items 4 and 5 without a new test. The `TestRenderReadmeGolden` doc comment at `render_test.go:267` still says "loom for an empty recipe"; it should say `tapestry` and name the profile. That stale comment is a minor extra.
- **NIT, consistency.** `render_test.go:267`. The comment says "loom for an empty recipe", but `defaultRecipe` is `tapestry` and the rows show `tapestry`. It predates the diff, but the diff changes this cell's contract and leaves the comment without the profile suffix. Fix: reword to "tapestry for an empty recipe, followed by ` · ` and the profile when the task names one".

No duplicated test was added; the diff changes one existing golden row only.

**Tool calls**
1. `grep -n "Profile:\|Recipe: \|tapestry" internal/boardengine/*_test.go`: to find every fixture with a profile or an empty recipe, which the manifest lacked.
