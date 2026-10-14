# Card 1 — external-test-unit-dir

**Create:**
- `plan:internal/planparser#externalTestPackageDir` -> `func externalTestPackageDir(unit string) string`

**Edit:**
- `internal/planparser#refUnit`
- `internal/planparser#CardTargetDirs`
- `internal/planparser#brokenFabricVocabularyRule`
- `internal/planparser_test#TestCardTargetDirs`
- `internal/websterengine#TestRenderBatchGate`
- `internal/websterengine#TestCardGateCommand`
- `internal/batcher_test#TestPeakContext`
- `internal/planparser/doc.go`

**Prosa:**
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`

**Intent:**
D1 and D2: an external test package's glyph unit maps to its package directory, so no derived gate names `./internal/x_test`.

`externalTestPackageDir` lives in `internal/planparser/targetdirs.go`.
It takes a Go unit string, either a glyph's `UnitPath` result or a handle's package half, and when the unit's last path segment ends in `_test` and carries no file extension (`hasFileExtension` false) it returns the unit with that `_test` suffix removed; any other unit comes back unchanged.
It is disk-free and unconditional: a real package directory whose own name ends in `_test` maps to its sibling, which its doc comment states as the bound (this repository has none).

`refUnit` applies it to the unit of every ref that parses as a glyph or `plan:` handle, right after `UnitPath`, so `CardTargetDirs` and `RefFile` both see the package directory.
A path ref that does not parse as a glyph, and a file unit, are untouched.
`CardTargetDirs`' doc comment gains one sentence: an external test package's `<dir>_test` unit maps to `<dir>`, so a card targeting both `internal/x#…` and `internal/x_test#…` yields one `internal/x` entry.
Webster's `gateScopeOf` (card gate and batch gate alike) and `internal/batcher`'s read-set estimate read `CardTargetDirs`, so neither changes code.

`brokenFabricVocabularyRule` replaces its `strings.TrimSuffix(unit, "_test")` with `externalTestPackageDir(unit)`; the owner check behaves as before, which the existing `test name in an owner package's external test package` row of `TestCheckFabricVocabularyName` keeps pinning.

Test rows:
- `TestCardTargetDirs` gains three rows: an Edit of `internal/a_test#TestRun` yields `{Dir: "internal/a", NamesGo: true}`; an Edit of `internal/a#Run` and `internal/a_test#TestRun` yields that one entry once; a plain directory path `internal/a_test`, which parses as no glyph, stays `{Dir: "internal/a_test"}`.
  A handle takes the glyph branch, which the existing `handle maps through its glyph` row pins.
- `TestRenderBatchGate` gains a row whose two cards edit `internal/a#Run` plus `internal/a_test#TestRun` and `internal/b_test#TestB`, with no file on disk: the package list names the root module's `./internal/a ./internal/b` and the gate is the single step `lyx gate test --tags integration ./internal/a ./internal/b`, in the format the table's first row shows minus its nested-module lines.
- `TestCardGateCommand` gains the subtest `an external test package gates on its package directory`: a card editing `internal/a#Run` and `internal/a_test#TestRun` yields `lyx gate test ./internal/a`, then the two lints.
- `TestPeakContext` gains the row `a package and its external test package weigh one directory`: `editCard(1, []string{"internal/a#Run", "internal/a_test#TestRun"})` weighs `startup + (2*3 + 2*4) + 7 + 7`; the old mapping counts a second package and tests entry, 14 more.

Docs: `internal/planparser/doc.go`'s Glyph Conversion Chokepoint section says `externalTestPackageDir` is the one sanctioned adjustment of a Go unit string, and the later passage claiming `Glyph.UnitPath` is "used only by validate.go's disk-existence checks" names `targetdirs.go`'s use and that adjustment too.

`PATTERN.md`'s `PATTERN-glyph-conversion-chokepoint` line becomes exactly this line, which names the exception and keeps the overview from growing: `lyx loom lint-pattern`'s headroom check refuses any growth, since the file sits at its headroom line, and the four calls stay named in the background.

```
- `PATTERN-glyph-conversion-chokepoint` — Glyph-path conversion: only `glyph`'s calls, plus `planparser`'s `_test` unit map. — [background](pattern/PATTERN-glyph-conversion-chokepoint.md)
```

`pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet: `planparser`'s `externalTestPackageDir` is the single sanctioned adjustment of a Go unit string, for quarry's external-test pseudo-unit `<dir>_test` only, which quarry's `docs/glyph.md` corner case documents as having no directory.

The commit body names #571, #577, #580 and #587.

**ImpactSummary:** every derived card and batch gate, and the batcher's estimate, name an external test package's directory instead of its `_test` pseudo-path; vocabname's owner check is unchanged.

**Commit:** 1: planparser: map an external test package's unit to its directory
