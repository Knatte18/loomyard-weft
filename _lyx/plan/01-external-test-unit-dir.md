# Card 1 — external-test-unit-dir

**Create:**
- `plan:internal/planparser#externalTestUnitDir` -> `func externalTestUnitDir(unit string) string`

**Edit:**
- `internal/planparser#refUnit`
- `internal/planparser#CardTargetDirs`
- `internal/planparser#brokenFabricVocabularyRule`
- `internal/planparser_test#TestCardTargetDirs`
- `internal/websterengine#TestCardGateCommand`
- `internal/batcher_test#TestPeakContext`
- `internal/planparser/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`

**Uses:**
- `internal/planparser_test#TestCheckFabricVocabularyName`

**Intent:** Closes #571, #577, #580 and #587 (D1, D2): a card targeting a member glyph on a Go external test package, such as `internal/x_test#TestY`, no longer yields the gate argument `./internal/x_test`.

`externalTestUnitDir` takes a Go unit string, either a glyph's `Glyph.UnitPath` result or a handle's package half, and returns it with a trailing `_test` removed when its last path segment ends in `_test` and has no file extension; any other unit comes back unchanged.
It is pure string work on the unit's last segment and reads no disk.
`refUnit` applies it to every glyph-derived unit, the `fromGlyph` branch (a glyph or a `plan:` handle), so `CardTargetDirs` maps `internal/x_test` to `internal/x`, and an `internal/x#Run` and an `internal/x_test#TestRun` target on one card merge into one `internal/x` entry.
Path refs and file units are untouched: a path ref never reaches the helper, and a file unit carries an extension.
`RefFile` reads the same `refUnit` and is unchanged in effect, since a package unit names no file.
`CardTargetDirs`' doc comment gains the sentence that an external test package's unit maps to its package directory.
`brokenFabricVocabularyRule` replaces its `strings.TrimSuffix(unit, "_test")` with a call to the helper; `TestCheckFabricVocabularyName`'s existing rows for `internal/configsync_test` and `internal/fabricengine_test` pin that the owner check behaves as before.
`cmd/lyx/constraintchokepoint_test.go` scans only `#`-trimming `TrimSuffix`, so the helper needs no exemption there.

Bound: the mapping is unconditional, so a real package directory whose own name ends in `_test` would map to its sibling; this repository has none.

Both production readers of `CardTargetDirs` follow without a code change: webster's `gateScopeOf` and `internal/batcher`'s `loadCard` read set, whose `package:` and `tests:` entries for `internal/x_test` now become `internal/x` and merge with an existing one.

Tests, all rows in existing tests:
- `TestCardTargetDirs`: a member glyph `internal/x_test#TestY` maps to `{Dir: "internal/x", NamesGo: true}`; `internal/x#Run` beside `internal/x_test#TestRun` yields one `internal/x` entry; a path ref `internal/y_test` stays `internal/y_test`.
- `TestCardGateCommand`: a subtest whose card edits `internal/x#Run` and `internal/x_test#TestRun` gets exactly `lyx gate test ./internal/x` plus the two lints, never `./internal/x_test`.
  The batch gate reads the same `gateScopeOf`, and its union across cards is already pinned by `TestRenderBatchGate`'s two-card row, so it takes no row of its own.
- `TestPeakContext`: a row `editCard(1, []string{"internal/a#Run", "internal/a_test#TestRun"})` wants `startup + (2*3 + 2*4) + 7 + 7`: two targets, the two test files of `internal/a`, and one package and one tests entry; the unmapped code adds a second package and tests pair for `internal/a_test`.

`PATTERN.md`'s entry becomes this line verbatim:

```
- `PATTERN-glyph-conversion-chokepoint` — Converting glyphs and paths: only `glyph`'s `Self`, `UnitPath`, `Parse` and `String`, plus `planparser`'s one external-test unit mapping. — [background](pattern/PATTERN-glyph-conversion-chokepoint.md)
```

`pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet naming `planparser`'s `externalTestUnitDir` as the single sanctioned adjustment of a Go unit string, for the external-test pseudo-unit only, citing quarry's `docs/glyph.md` corner case that an external test package's unit `<dir>_test` names no directory.
`internal/planparser/doc.go`'s Glyph Conversion Chokepoint section names the same exception, and its later paragraph saying `Glyph.UnitPath` is "used only by validate.go's disk-existence checks" is corrected to name `targetdirs.go`'s `refUnit` too, with its output passed through `externalTestUnitDir`.

**ImpactSummary:** One mapping inside planparser changes the directory every glyph on an external test package yields, which fixes webster's card and batch gates and the batcher's read-set estimate with no change to either caller.

**Commit:** `1: Gate steps name an external test package's own directory`
