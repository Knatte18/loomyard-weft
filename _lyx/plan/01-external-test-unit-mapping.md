# Card 1 — External-test glyph units map to their package directory

**Create:**
- `plan:internal/planparser#goPackageUnit` -> `func goPackageUnit(unit string) string`

**Edit:**
- `internal/planparser#refUnit`
- `internal/planparser#CardTargetDirs`
- `internal/planparser#brokenFabricVocabularyRule`
- `internal/planparser_test#TestCardTargetDirs`
- `internal/websterengine#TestRenderBatchGate`
- `internal/batcher_test#TestPeakContext`
- `internal/planparser/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`

**Intent:**
D1 and D2.
Quarry names a Go external test package's unit `<dir>_test`, a pseudo-path with no directory, and `Glyph.UnitPath` returns it unchanged, so a card targeting `internal/x_test#TestY` yields the gate argument `./internal/x_test` in webster's card and batch gates (#571, #577, #580, #587).

`goPackageUnit`, in `internal/planparser/targetdirs.go`, takes a Go unit string, either a glyph's `UnitPath` result or a handle's package half.
When the unit's last path segment ends in `_test` and has no file extension, it returns the unit with that suffix removed; any other unit, a file unit included, it returns unchanged.
Its doc comment states the bound: the mapping is disk-free and unconditional, so a real package directory whose own name ends in `_test` maps to its sibling.

`refUnit` applies `goPackageUnit` to the unit every glyph and `plan:` handle yields, after `UnitPath`; a path ref is returned untouched.
`RefFile` reads `refUnit` too and is unaffected, since `goPackageUnit` leaves a file unit alone.
`CardTargetDirs`'s doc comment says a glyph's external-test unit maps to its package directory, so a card targeting `internal/x#…` and `internal/x_test#…` yields one `internal/x` entry.
Both production readers change with it and need no edit of their own: webster's `gateScopeOf` and `internal/batcher`'s read-set estimate.
`brokenFabricVocabularyRule` replaces its `strings.TrimSuffix(unit, "_test")` with `goPackageUnit(unit)`; the unit there is a handle's package half, which never carries an extension, so the owner check behaves as before, and `TestCheckFabricVocabularyName`'s existing external-test rows pin that unchanged.

Tests, all rows of existing tables:
- `TestCardTargetDirs` gains rows: a member glyph `internal/a_test#TestRun` maps to `{Dir: "internal/a", NamesGo: true}`; `internal/a#Run` beside `internal/a_test#TestRun` in one Edit group yields the single entry `internal/a`; a directory path ref `internal/a_test` stays `internal/a_test`, since path refs are untouched.
  A handle needs no row of its own, since `refUnit` sends a handle's body down the same glyph branch.
- `TestRenderBatchGate` gains a row: one card editing `internal/a/a.go#` and another editing `internal/a_test#TestA` render the root module's packages as `./internal/a` alone and the batch gate `lyx gate test --tags integration ./internal/a`.
  Card gates come from the same `gateScopeOf`, so this row pins both gate forms.
- `TestPeakContext` gains a row: a card whose Edit targets are `internal/a#Run` and `internal/a_test#TestRun` counts `package:internal/a` and `tests:internal/a` once and the two test files of `internal/a` once, so its peak is `startup + 2*3 + 2*4 + 7 + 7`; with the bug it counts a second package and tests entry for `internal/a_test`.

Docs, D2:
- `PATTERN.md`'s `PATTERN-glyph-conversion-chokepoint` line becomes, verbatim:

```markdown
- `PATTERN-glyph-conversion-chokepoint` — Converting glyphs and paths: only `glyph`'s `Self`, `UnitPath`, `Parse` and `String`, plus `planparser`'s external-test unit mapping. — [background](pattern/PATTERN-glyph-conversion-chokepoint.md)
```

- `pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet naming `goPackageUnit` in `internal/planparser` as the single sanctioned adjustment of a Go unit string, for the external-test pseudo-unit only, because quarry's `docs/glyph.md` documents that pseudo-unit as naming no directory, so some caller must map it.
- `internal/planparser/doc.go`'s Glyph Conversion Chokepoint section, and its later sentence naming `Glyph.UnitPath` as the sole glyph-to-path call, name `goPackageUnit` as the one adjustment applied to a `UnitPath` result.

**ImpactSummary:** every card and batch gate and every batcher read-set entry derived from an external-test glyph now names its package directory; the fabric-vocabulary owner check is unchanged.

**Commit:** 1: map an external-test glyph unit to its package directory
