```yaml
lens: code-correctness
outcome: reported
```

Task: check the correctness of every function, branch and error path the diff changes, using the code-correctness lens.

**Cases run**

- **`internal/shedcli/table.go`: `names` and the `sort` import removed.** I checked whether anything in the file as the diff leaves it still uses `sort`. Nothing does (`table.go:7-15`, `:97-106`). The only map reads left are `recipes[name]` in `lookup` (`table.go:102`). The production callers of the table go through `lookup` alone (`cli.go:255`, `seed.go:72`). Correct.
- **`TestRecipes_KeySet`, literal row (`table_test.go:43`), on the real map.** `slices.Sorted(maps.Keys(recipes))` over the 9 keys at `table.go:48-95` gives `[arras batten darn hardener motif pawl raddle sampler tapestry]`. `slices.Equal` against the literal is true, so the row passes. Correct.
- **`TestRecipes_KeySet`, shedrun row (`table_test.go:44`).** `RecipeNames()` returns the same 9 names in the same sorted order (`shedrun/seed.go:99`). `slices.Equal` is true. Correct.
- **Boundary: an empty `recipes` map.** `slices.Sorted` of an empty sequence returns nil. `slices.Equal(nil, want)` is false for a 9-element `want`, so `Errorf` fires. The old code reached the same failure through `Fatalf` on the length check. Correct.
- **Boundary: an extra or a missing key.** The lengths differ, so `slices.Equal` is false and the row fails. The old length check failed on the same input. Correct.
- **Boundary: `RecipeNames()` in an order that isn't sorted.** The old code compared element by element against a sorted `got`, and `slices.Equal` compares in order too, so both fail. The check is no looser. Correct.
- **Failure-path reporting.** The old code was `Fatalf` on a length mismatch, then `Errorf` per index. The new code is one `Errorf` that prints both whole slices. No later assertion depends on it, so `Errorf` without `Fatal` follows the testing convention and drops nothing. Correct.
- **Imports and toolchain.** `go.mod` declares go 1.26, and `maps.Keys` plus `slices.Sorted` are in the standard library there. `sort` is still used by `TestTable_NoInitNoRegisterSeam` (`table_test.go:140`, `:144`), so keeping its import is right. Correct.

**Findings:** none.

**Tool calls:** none. The inherited context already held everything this lens needed.
