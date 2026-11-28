```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: tracing the diff's changes to the goal points of the decision record, which is the answer key here (there is no plan).

**Decisions (goal points of the board entry under `## Goal`; the record has no `Added after Discussion` entries):**
- G1: remove `names` with its test, or give it a production caller.
- G2: edit the file header comment that names the `names` accessor.

**Changes:**
- C1: `internal/shedcli/table.go` deletes `names` and its `sort` import. Traces to G1, which offers deleting as one of its two options.
- C2: `internal/shedcli/table.go:2` header now names only the lookup accessor. Traces to G2.
- C3: `internal/shedcli/table_test.go`, `TestRecipes_KeySet` reads `slices.Sorted(maps.Keys(recipes))` in place of `names()`, and the file adds a `maps` import.
  - Forced by C1, since the test called `names`.
  - G1 says "remove `names` with the test of it". The diff keeps the test and moves it off `names`.
  - The test pins the recipes key set against the literal list and `shedrun.RecipeNames()`, which is a recipes-table behaviour, not a behaviour of `names`. Its `testtiming:keep` line (`table_test.go:35`) says no other test covers it.
  - So keeping it falls within G1's intent, and C3 is no scope finding.
- C4: `internal/shedcli/table_test.go:1-2` file header drops `names` and the "two accessors" wording. Forced by C1, since the old header went stale.

**Per decision:**
- G1: built by C1, with C3 forced.
- G2: built by C2, with C4 forced.

**Findings:** none. Every change traces to G1 or G2 or is forced by one, and each goal point is built. The commit carries no `Pattern-Departure:` trailer, and none is needed.

Tool calls: none; everything needed was already in context.
