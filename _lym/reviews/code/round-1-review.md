---
verdict: APPROVED
---

The range `3fb0787e3..HEAD` is one commit, `2f7d8bd82`, which deletes `internal/shedcli.names` with its `sort` import, rewrites `TestRecipes_KeySet` to read the map's sorted keys itself, and updates both file headers.
The amended board entry asks to remove `names` or give it a production caller, and to edit the header that names it; the diff does both, by deletion.
Neither the four lenses nor the handler's own review found a defect.

Focus entries, each driven:

- `TestRecipes_KeySet`: `slices.Equal(slices.Sorted(maps.Keys(recipes)), tt.want)` fails on every input the old length check and per-index loop failed on: a missing key, an extra key, an empty map and an out-of-order expectation.
  The `Fatalf` that became an `Errorf` guarded no later assertion, so no limit is looser.
  The name, the doc comment and the `//testtiming:keep` reason still describe what the test checks.
- Stale descriptions: no comment, doc or spec under `internal`, `cmd`, `docs`, `pattern` or `PATTERN.md` still names the `names` accessor or two accessors.
  The new `table.go:2` header matches the code: `lookup` is the one production reader of `recipes`, called from `cli.go:255` and `seed.go:72`.
  The new `table_test.go:1-2` header matches the file, which reads `recipes` and calls `lookup`.
- `_lym/landing/summary.md`: at the base, `table_test.go` was the only caller of `names`.
  `lookup`'s error lists `shedrun.OfferedRecipeNames()`, which leaves out arras, so the claim that no production list needs the full armed set holds.
  A `two-way` door fits: the diff removes an unexported function and changes no record format, config meaning, data or outward push.
  The `shedcli` blast radius covers everything the diff can break.

## Rejected

- None: no lens raised a finding.
