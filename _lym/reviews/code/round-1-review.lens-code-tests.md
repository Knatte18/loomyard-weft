```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that pins it.

**Per-behaviour list**

1. **`names` removed from `internal/shedcli/table.go`, so every production function in the package is reached from a main package.**
   Pinned by `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` (`cmd/lym/testonlycode_linux_test.go`).
   The decision record's `## Goal` names this test as the one that fails at `3fb0787e3` on `names`, so it fails again if an unreached function comes back.
   The diff removes no other behaviour: at the base, `names` was read only by `table_test.go:56`. Nothing in `cli.go` or `seed.go` reads `recipes`; both reach it through `lookup` (`cli.go:255`, `seed.go:72`).
2. **`TestRecipes_KeySet` rewritten (`table_test.go:49-52`): the table's sorted key set must equal the literal list and `shedrun.RecipeNames()`.**
   The test pins itself, and its rows still fail on every break the old body caught:
   - `slices.Equal` checks length and every element, as the old length check plus per-index loop did.
   - Sorting moved from `names` into `slices.Sorted(maps.Keys(recipes))`, which does the same sort.
   - The expected values are still independent of the code under test: row 1 is a literal (`table_test.go:43`), and row 2 is `shedrun.RecipeNames()`, a literal in another package (`internal/shedrun/seed.go:99`).
   - The old `t.Fatalf` on a length mismatch became a single `t.Errorf`. Nothing ran after that assertion, so no limit is looser.
   - The test's name, doc comment (`table_test.go:31-33`) and `//testtiming:keep` reason (`table_test.go:35`) still match what it checks.
3. **File header comments changed (`table.go:2`, `table_test.go:1-2`).**
   These are static prose, so they are exempt and need no test.

**Findings**

None.

**Tool calls**

None. Everything this lens needed was already in my inherited context.
