# Card 1 — external-test-unit-dir

**Create:**
- `plan:internal/planparser#goPackageDir` -> `func goPackageDir(unit string) string`

**Edit:**
- `internal/planparser#targetDir`
- `internal/planparser#brokenFabricVocabularyRule`
- `internal/planparser#CardTargetDirs`
- `internal/planparser_test#TestCardTargetDirs`
- `internal/planparser/doc.go`
- `internal/websterengine#TestCardGateCommand`
- `internal/batcher_test#TestPeakContext`
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`

**Intent:** D1 and D2: an external-test glyph unit maps to its package directory.

Add `goPackageDir` to `internal/planparser/targetdirs.go`: it takes a Go unit string, either a glyph's `Glyph.UnitPath` result or a handle's package half, and when its last path segment ends in `_test` and has no file extension (`hasFileExtension` is false), returns it with that `_test` suffix removed; any other string is returned unchanged.
Its doc comment states the bound: the mapping is disk-free and unconditional, so a real Go package directory whose own name ends in `_test` maps to its sibling directory, and this repository has none.

`targetDir` applies it to every glyph-derived unit, that is when `refUnit` reports `fromGlyph` true, before `unitDir`; path refs and file units are untouched (a file unit carries an extension, so `goPackageDir` leaves it unchanged).
`CardTargetDirs` needs no code change beyond its doc comment, which gains one sentence: an external-test glyph unit `<dir>_test` maps to `<dir>`, so a card targeting both `internal/x#…` and `internal/x_test#…` yields one `internal/x` entry.
`RefFile` is unchanged.

`brokenFabricVocabularyRule` replaces its `strings.TrimSuffix(unit, "_test")` with `goPackageDir(unit)`, keeping its comment; behaviour is unchanged for every external-test unit, which the existing `TestCheckFabricVocabularyName` rows on `plan:internal/configsync_test#…` and `plan:internal/fabricengine_test#…` keep pinning.

Tests, rows only:
- `TestCardTargetDirs` gains rows: "external test handle maps to its package directory" (`plan:internal/a_test#TestNew` under Create -> `{Dir: "internal/a", NamesGo: true}`); "package and its external test share one entry" (`internal/a#Run` and `internal/a_test#TestRun` in one Edit group -> one `{Dir: "internal/a", NamesGo: true}`).
- `TestCardGateCommand` gains a subtest: a card whose Edit group targets `internal/x#Run` and `internal/x_test#TestRun` yields `lyx gate test ./internal/x` once, followed by the two lints, never `./internal/x_test`.
- `TestPeakContext` gains a row: a card editing `internal/a#Run` and `internal/a_test#TestRun` counts the `internal/a` package and tests entries once and the two `internal/a` test files once: `startup + 2*3 + 2*4 + 7 + 7`.

Docs in the same commit:
- `internal/planparser/doc.go` states, where it describes target directories, that an external-test glyph unit maps to its package directory through `goPackageDir`, the one sanctioned adjustment of a Go unit string.
  Its `# The Glyph Conversion Chokepoint Invariant` section, which says `Glyph.UnitPath` is the only glyph-to-path call, names `goPackageDir` as that invariant's one exception, for the external-test pseudo-unit only.
- `PATTERN.md`'s `PATTERN-glyph-conversion-chokepoint` line becomes, verbatim:

```
- `PATTERN-glyph-conversion-chokepoint` — Converting glyphs and paths: only `glyph`'s `Self`, `UnitPath`, `Parse` and `String`, plus `planparser`'s one external-test unit mapping. — [background](pattern/PATTERN-glyph-conversion-chokepoint.md)
```

- `pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet: `planparser`'s `goPackageDir` is the single sanctioned adjustment of a Go unit string, for the external-test pseudo-unit only: quarry names a Go external test package's unit `<dir>_test`, a pseudo-path with no directory (quarry's `docs/glyph.md`, its external-test corner case), and `goPackageDir` maps it to `<dir>`.
  The opening sentence "loomyard performs no glyph-to-path or path-to-glyph conversion of its own" gains "beyond that one mapping".

`internal/batcher/doc.go` names no test directory, so it is not changed.

**ImpactSummary:** `targetDir` feeds `CardTargetDirs`, whose two production readers, webster's `gateScopeOf` and batcher's read-set estimate, now merge `internal/x_test` into `internal/x`; `brokenFabricVocabularyRule` keeps its behaviour.

**Commit:** `1: map an external-test glyph unit to its package directory (#571, #577, #580, #587)`
