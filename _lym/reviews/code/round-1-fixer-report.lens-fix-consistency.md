```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for sibling or twin text that still disagrees with the change.

**Findings**

1. **LOW**: `internal/reportengine/doc.go:25` ("the seed names the loom or the darn recipe").
   - This is a twin of the recipe vocabulary change. The code at `internal/reportengine/runs.go:24` is `runRecipes = {shedrun.RecipeTapestry, shedrun.RecipeDarn}`, so the seed value is now `tapestry`, not `loom`.
   - The comment is stale. F3 and F5 fixed the same kind of statement elsewhere, but this file was not in the review's list.

2. **LOW**: `internal/loomcli/arm.go:197` and `internal/loomcli/peers.go:88` (not changed).
   - Both read `TapestryRetiredRows()`, so the new `Loom-Preflight` entry reaches them.
   - `internal/landingshed/handback.go:13` and `internal/landingshed/parentalign.go:24` handle the retired `Merge-In` name on their own, because batten and the arbiter read a child's status before any loom verb renames it. They have no `Loom-Preflight` counterpart.
   - I found no code reading `Loom-Preflight` from a child status. A halt stop at the old preflight name would not be read as a stop there, but `Preflight` is never a handback stop row, so I see no defect. This is a note, not a defect.

No other twin found. The F1 doc text (`loomrecipe.go`, `doc.go`, `tapestry-recipe.yaml`) agrees with the map and the new test row. `contracts/recipes/darn-recipe.yaml:52` lists `Merge-In` for darn and is unaffected. The remaining `loom` recipe mentions I grepped are `lym loom` verbs or config-module names and are correct.

**Calls made**
1. A grep for `Merge-In` references across Go, markdown and yaml: to find twins of the retired-row text.
2. A grep for `loom` recipe prose and `loom` seed text: to find recipe-value mentions the review missed.
3. A read of `internal/reportengine/doc.go:25` and `loom-status-spec.md:160`, plus a grep for `"loom"` in `reportengine`: my context lacked those files.
4. A grep for the recipe constants in `reportengine`: to confirm that the code uses `tapestry`.
