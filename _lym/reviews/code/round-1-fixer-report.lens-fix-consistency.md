```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- The spec now names `tapestry.yaml` and `tapestry.implementer` and no longer mentions `loom.implementer`. That matches `contracts/recipes/tapestry-recipe.yaml:386` (`implementer_setting: tapestry.implementer`) and the `tapestry.implementer` strings in `internal/shedrecipe/entries_implementer.go`.
- I found no other `loom.implementer` reference in the Go, markdown or yaml files outside `_lym`.
- The line says "the recipe's Webster row", and `tapestry-recipe.yaml:9` lists Webster as a row of the tapestry recipe. The row table below it names `webster` as the only implementer, so the two agree.
- Outside the changed text, `internal/shedrecipe/entries_implementer_test.go:90` has a test fixture error string `"loom.yaml unreadable"`. It is an arbitrary lookup-error message, not a claim about which file holds the key, and the fix did not touch or reach it.

Tool calls:
1. `grep` for `loom.implementer` and `tapestry.implementer` across the repo: my context did not show whether twins of the removed sentence survive elsewhere.
