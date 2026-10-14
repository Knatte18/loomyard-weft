# Card 1 — external-test-unit-dir

**Create:**
- `plan:internal/planparser#externalTestPackageDir` -> `func externalTestPackageDir(unit string) string`

**Edit:**
- `internal/planparser#refUnit`
- `internal/planparser#CardTargetDirs`
- `internal/planparser#RefFile`
- `internal/planparser#brokenFabricVocabularyRule`
- `internal/planparser_test#TestCardTargetDirs`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine#TestRenderBatchGate`
- `internal/batcher_test#TestPeakContext`
- `internal/planparser/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`

**Intent:** D1 and D2: an external-test glyph unit maps to its package directory, so no derived gate names `./internal/x_test` (#571, #577, #580, #587).

Add `externalTestPackageDir` to `internal/planparser/targetdirs.go`.
It takes a Go unit string, either a glyph's `UnitPath` result or a handle's package half, and when the unit's last slash-separated segment ends in `_test` and has no file extension (`hasFileExtension` is false), returns the unit with that `_test` suffix removed; any other unit is returned unchanged.
Its doc comment names it as the one sanctioned adjustment of a Go unit string, for quarry's external-test pseudo-unit (`<dir>_test`, which quarry's `docs/glyph.md` documents as having no directory), and states the bound: the mapping is disk-free and unconditional, so a real package directory whose own name ends in `_test` maps to its sibling; this repository has none.

`refUnit` applies it to the unit of every glyph-derived ref (glyph or `plan:` handle) on the line that returns `unit, true, true`; a path ref stays untouched, and a file unit passes through unchanged because its last segment carries an extension.
Since `CardTargetDirs` and `RefFile` both go through `refUnit`, Webster's `gateScopeOf` (card and batch gates alike) and `internal/batcher`'s read-set estimate both see `internal/x` for an `internal/x_test#…` target, and an entry for both `internal/x#…` and `internal/x_test#…` merges into one directory through `CardTargetDirs`'s existing first-seen dedup.
`RefFile` is unaffected in practice: an external-test unit has no file extension, so it never names a file.
The doc comments of `CardTargetDirs` and `RefFile`, which say a glyph or `plan:` handle maps through `Glyph.UnitPath`, gain that the unit then passes through `externalTestPackageDir`; their code does not change.

`brokenFabricVocabularyRule` replaces `strings.TrimSuffix(unit, "_test")` with `externalTestPackageDir(unit)`, so its owner check behaves as before for an external-test unit; `TestCheckFabricVocabularyName`'s existing `configsync_test` and `fabricengine_test` rows pin that unchanged, so it takes no new row.

Tests, rows only:
- `TestCardTargetDirs`: one row "external test package glyph and handle map to their package directory" with an `Edit` group targeting `internal/a_test#TestRun` and a `Create` group targeting `plan:internal/b_test#TestNew`, wanting `{Dir: "internal/a", NamesGo: true}` then `{Dir: "internal/b", NamesGo: true}`.
  The merge of `internal/a` and `internal/a_test` into one entry is the existing first-seen dedup, pinned at the consumer rows below.
- `TestCardGateCommand`: a subtest "an external test package is gated as its package directory" whose card edits `internal/a#Run` and `internal/a_test#TestRun`, wanting `lyx gate test ./internal/a` once, then the two lints; it fails today with `./internal/a ./internal/a_test`.
- `TestRenderBatchGate`: a row "an external test package joins its package directory in the batch gate" whose two cards edit `internal/a#Run` and `internal/a_test#TestRun`, wanting `./internal/a` once in the root module's package list and in the `lyx gate test --tags integration ./internal/a` step.
- `TestPeakContext`: a row "a package and its external test package share one read-set entry" whose card edits `internal/a#Run` and `internal/a_test#TestRun`, wanting `startup + 2*3 + 2*4 + 7 + 7`: two targets, the two test files of `internal/a` once, and one package and one tests entry; it fails today because `internal/a_test` adds a second package and tests entry.

Docs: `internal/planparser/doc.go`'s "Glyph Conversion Chokepoint Invariant" section names `externalTestPackageDir` as its one sanctioned unit adjustment.
`PATTERN.md`'s `PATTERN-glyph-conversion-chokepoint` line becomes:

```
- `PATTERN-glyph-conversion-chokepoint` — Converting glyphs and paths: only `glyph`'s `Self`, `UnitPath`, `Parse` and `String`, plus `planparser`'s one external-test unit mapping. — [background](pattern/PATTERN-glyph-conversion-chokepoint.md)
```

`pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet: `planparser`'s `externalTestPackageDir` is the single sanctioned adjustment of a Go unit string, for the external-test pseudo-unit only, which quarry's `docs/glyph.md` documents as a unit with no directory; it trims the `_test` suffix of a last segment without an extension, and nothing else adjusts a unit.

**ImpactSummary:** `refUnit` feeds `CardTargetDirs` and `RefFile`, so Webster's card and batch gates and the batcher's estimate change only for `_test` units; the vocabulary check keeps its behavior.

**Commit:** `1: Map an external-test glyph unit to its package directory (#571, #577, #580, #587)`
