```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens on the diff that removes `internal/shedcli.names`. The diff leaves nothing in the affected set wrong, so this lens has no findings.

**Change 1: `names()` and its `sort` import deleted from `internal/shedcli/table.go`.**
- **Callers:** the only caller was `table_test.go:56` at the review base, and the diff rewrites that line. No `names()` call is left in `internal/shedcli` or `cmd`. Covered.
- **Production readers of `recipes`:** only `lookup` reads the map (`table.go:101-106`). It is reached from `cli.go:255` and `seed.go:72`. Nothing else in the package needed `names`. Covered.
- **The test-only-code gate's allowlist:** `testOnlyCodeAllowed` in `cmd/lym/testonlycode_linux_test.go:50` has no `internal/shedcli.names` entry. Its shed entries are `internal/loomrecipe.RecipeEngines` and `internal/shedrecipe.Names`. That allowlist fails on an entry that matches no function (`RequireNoStale`, line 106), so a leftover entry would have broken the gate. There is none, so nothing is left stale. Covered.
- **The `sort` import in `table.go`:** nothing else in the file uses it (read whole, `table.go:1-106`). Covered.
- **Rule documents (PATTERN.md, `pattern/`, `cmd`, `internal`, `docs`):** a search for `shedcli.names` and `shedcli/names` matched nothing. PATTERN-test-only-code requires every production function to be reachable from a `main`, and removing `names` satisfies it. The directive declares no PATTERN changes, so there is nothing to check under "Declared PATTERN changes landed". Covered.

**Change 2: the file headers.**
- **`table.go:2`** now says the recipes are "reached only through the lookup accessor". This matches the code, since `lookup` is the one production reader of `recipes`. Covered.
- **`table_test.go:1-2`** now says the file covers the map and its lookup accessor, and "reads recipes and calls lookup alone". This matches the code: the file reads `recipes` at lines 49 and 102 and calls `lookup` at line 80. Covered.
- **The package doc in `doc.go:11-13`** describes the table, not its accessors, and needs no edit. Covered.
- **A search for "accessor" in `internal/shedcli`** finds only the two updated headers. Covered.

**Change 3: `TestRecipes_KeySet` now reads `slices.Sorted(maps.Keys(recipes))`.**
- **The doc comment (`table_test.go:31-35`)** and its `//testtiming:keep` reason name no accessor, so both still hold. Covered.
- **The `shedrun.RecipeNames()` row:** the function returns its names in sorted order (`internal/shedrun/seed.go:98-99`), so comparing it against the sorted keys is still valid. Covered.
- **The `maps` import:** the go.mod directive is `go 1.26`, and `maps.Keys` needs 1.23, so it is supported. Covered.

**Commit messages and departures:** the range's one commit, `2f7d8bd82`, has no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed. Covered.

Tool calls:
1. A search for `shedcli.names` in `cmd`, `internal`, `pattern`, `PATTERN.md` and `docs`, plus the allowlist lines of `cmd/lym/testonlycode_linux_test.go`. My context lacked the test-only-code gate's allowlist.
2. The `shed` entries of that allowlist and a repeat search for `names()` call sites. I needed to confirm no `shedcli.names` allowlist entry was left stale.
