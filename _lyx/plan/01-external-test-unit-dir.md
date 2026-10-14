# Card 1 — external-test-unit-dir

**Create:**
- `plan:internal/planparser#goPackageDir` -> `func goPackageDir(unit string) string`

**Edit:**
- `internal/planparser#CardTargetDirs`
- `internal/planparser#refUnit`
- `internal/planparser#brokenFabricVocabularyRule`
- `internal/planparser_test#TestCardTargetDirs`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine#TestRenderBatchGate`
- `internal/batcher_test#TestPeakContext`
- `internal/planparser/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`

**Intent:** D1 and D2: an external-test glyph unit maps to its package directory, so no derived gate names `./internal/x_test`.

Add `goPackageDir` in `internal/planparser/targetdirs.go`.
It takes a Go unit string, a glyph's `UnitPath` result or a handle's package half, and when the unit's last path segment ends in `_test` and has no file extension, returns the unit with that suffix removed; any other unit comes back unchanged.
The mapping reads no disk and is unconditional, so a real package directory whose own name ends in `_test` would map to its sibling; this repository has none.
Its doc comment names it as the single sanctioned adjustment of a Go unit string, for quarry's external-test pseudo-unit only, citing quarry's `docs/glyph.md` corner case.

`refUnit` applies `goPackageDir` to every glyph-derived unit (the `UnitPath` branch, handles included), so `CardTargetDirs` maps `internal/x_test#TestY` and `plan:internal/x_test#TestY` to `internal/x`.
Path refs and file units are untouched, and `RefFile` keeps its answers, since an external-test unit has no file extension either way.
`CardTargetDirs`' first-seen merge then makes `internal/x#…` and `internal/x_test#…` on one card a single `internal/x` entry.
`CardTargetDirs`' doc comment, which says a glyph or handle maps through `Glyph.UnitPath`, adds that an external-test unit maps to its package directory; its body is unchanged.
Both production readers change with it and need no code edit: Webster's `gateScopeOf` renders `./internal/x`, and `internal/batcher`'s read-set estimate keys `package:internal/x` and `tests:internal/x` once.

`brokenFabricVocabularyRule` replaces its `strings.TrimSuffix(unit, "_test")` with `goPackageDir(unit)`, and keeps its comment's reason; its owner check behaves as before, which `TestCheckFabricVocabularyName`'s existing external-test rows (`internal/configsync_test`, `internal/fabricengine_test`) pin unchanged.

Tests, rows only:
- `TestCardTargetDirs` gains a row "external-test member glyph maps to its package directory" (`internal/a_test#TestRun` → `{Dir: "internal/a", NamesGo: true}`), a row for the handle `plan:internal/a_test#TestRun` with the same result, and a row with `internal/a#Run` and `internal/a_test#TestRun` on one Edit group yielding the one entry `{Dir: "internal/a", NamesGo: true}`.
- `TestCardTargetDirs` gains a row whose ref is the path `internal/a_test/a.go`, mapping to `internal/a_test`, pinning that a path ref is untouched.
- `TestCardGateCommand` gains a subtest: a card editing `internal/a#Run` and `internal/a_test#TestRun` gets `lyx gate test ./internal/a` once, then the two lints.
- `TestRenderBatchGate` gains a row: a two-card batch, card 1 editing `internal/a#Run` and card 2 editing `internal/a_test#TestRun`, renders `./internal/a` once in the package list and in `lyx gate test --tags integration ./internal/a`.
- The gate and estimate rows pin the acceptance criterion's named consumers, the card gate, the batch gate and the batcher's estimate, which read `CardTargetDirs` with no code change of their own; the `TestCardTargetDirs` merge row alone would not catch one of them stopping going through it.
- `TestPeakContext` gains a row "an external-test target shares its package's directory": one card targeting `internal/a/a.go` and `internal/a_test#TestA` whose peak counts `package:internal/a`, `tests:internal/a` and the two `internal/a` test files once, beside the messages of its two targets.

Docs in the same commit:
- `internal/planparser/doc.go`'s glyph-chokepoint paragraph (the `glyph.Self` … `Glyph.UnitPath` lines) names `goPackageDir` as the one sanctioned unit adjustment, and its `CardTargetDirs` mention says an external-test unit maps to its package directory.
- `PATTERN.md`'s entry becomes, verbatim:

```
- `PATTERN-glyph-conversion-chokepoint` — Converting glyphs and paths: only `glyph`'s `Self`, `UnitPath`, `Parse` and `String`, plus `planparser`'s one external-test unit mapping. — [background](pattern/PATTERN-glyph-conversion-chokepoint.md)
```

- `pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet: `planparser`'s `goPackageDir` is the single sanctioned adjustment of a Go unit string, for the external-test pseudo-unit `<dir>_test` only, which quarry's `docs/glyph.md` documents as having no directory; it trims that suffix and nothing else.

`cmd/lyx/constraintchokepoint_test.go` scans only `#`-trimming `TrimSuffix`, so `goPackageDir` needs no exemption there.

**ImpactSummary:** One planparser mapping changes every glyph-derived target directory, which Webster's card and batch gates and the batcher's estimate read unchanged; path refs, file units and the vocabulary check's verdicts stay as they are.

**Commit:** `1: external-test-unit-dir`
