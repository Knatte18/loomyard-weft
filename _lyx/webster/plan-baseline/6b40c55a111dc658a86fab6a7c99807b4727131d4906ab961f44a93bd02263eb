# Card 1 — external-test-unit-dir

**Create:**
- `internal/planparser#goPackageDir`

**Edit:**
- `internal/planparser#CardTargetDirs`
- `internal/planparser#targetDir`
- `internal/planparser#brokenFabricVocabularyRule`
- `internal/planparser/targetdirs_test.go`
- `internal/planparser/doc.go`
- `internal/websterengine#TestCardGateCommand`
- `internal/batcher/estimate_test.go`
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`
- `contracts/specs/loom-plan-spec.md`

**Intent:** D1 and D2: an external-test glyph unit maps to its package directory, so no derived gate names `./internal/x_test`.

`goPackageDir` goes in `internal/planparser/targetdirs.go`.
It takes a Go unit string, a glyph's `UnitPath` result or a handle's package half, and when its last path segment ends in `_test` and has no file extension (`hasFileExtension` false) it returns the unit with that `_test` suffix removed; any other unit comes back unchanged.
Its doc comment names it the one sanctioned adjustment of a Go unit string under `PATTERN-glyph-conversion-chokepoint`, for quarry's external-test pseudo-unit, which has no directory of its own.
It reads no disk: a real package directory whose own name ends in `_test` maps to its sibling, and this repository has none.

`targetDir` applies it to every glyph-derived unit (`fromGlyph` true from `refUnit`) before `unitDir`, so a glyph or `plan:` handle on `internal/x_test` yields the `TargetDir` `internal/x`, and a card targeting both `internal/x#…` and `internal/x_test#…` yields one `internal/x` entry, since `CardTargetDirs` keys entries by directory.
A path ref and a file unit are untouched: `RefFile` keeps reading `refUnit` directly.
`CardTargetDirs`'s doc comment adds that an external test package's unit maps to its package directory.
Both production readers follow with no code change: webster's `gateScopeOf` renders `./internal/x` once through `appendUnique`, and `internal/batcher`'s `loadCard` keys `package:internal/x` and `tests:internal/x` once.

`brokenFabricVocabularyRule` replaces `strings.TrimSuffix(unit, "_test")` with `goPackageDir(unit)`; its owner check behaves as before, which `TestCheckFabricVocabularyName`'s existing row "test name in an owner package's external test package" pins.

Tests:
- `TestCardTargetDirs` (in `internal/planparser/targetdirs_test.go`) gains rows: "external test package glyph maps to its package directory" (`internal/a_test#TestRun` in an Edit group, want `{Dir: "internal/a", NamesGo: true}`) and "package and its external test package merge into one directory" (`internal/a#Run` and `internal/a_test#TestRun`, want one `internal/a` entry).
  Each fails today with an `internal/a_test` entry.
  A handle reaches the same mapping, which the existing row "handle maps through its glyph" pins, and the existing rows on `.go` file units and directory paths pin that those stay untouched.
- The acceptance criterion names the gates and the batcher's estimate themselves, so one row in each pins those readers end to end.
- `TestCardGateCommand` gains the subtest "external test package maps to its package directory": a card targeting `internal/a#Run` and `internal/a_test#TestRun` renders `lyx gate test ./internal/a` once, followed by the two lints.
- `TestPeakContext` (in `internal/batcher/estimate_test.go`) gains the row "a package and its external test package count one directory": `editCard(1, []string{"internal/a#Run", "internal/a_test#TestRun"})`, want `startup + (2*3 + 2*4) + 7 + 7`; the bug adds a second `package:`/`tests:` pair of 14.

Docs:
- `PATTERN.md`'s `PATTERN-glyph-conversion-chokepoint` line becomes: "Converting glyphs and paths: only `glyph`'s `Self`, `UnitPath`, `Parse` and `String`, plus `planparser`'s one external-test unit mapping."
- `pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet naming `planparser`'s `goPackageDir` as the single sanctioned adjustment of a Go unit string, for the external-test pseudo-unit only, citing the corner case in quarry's `docs/glyph.md`: quarry names an external test package's unit `<dir>_test`, a pseudo-path with no directory, and `Glyph.UnitPath` returns it unchanged, so some caller must map it.
  Its opening sentence keeps "no conversion of its own" true by naming that exception.
- `internal/planparser/doc.go`'s Glyph Conversion Chokepoint section and its glyphref paragraph name `goPackageDir` beside `Glyph.UnitPath`.
- `contracts/specs/loom-plan-spec.md`'s sentence that `internal/planparser` performs exactly one glyph conversion in each direction adds `goPackageDir`'s external-test unit mapping as the one sanctioned adjustment.

**ImpactSummary:** one new helper applied inside `targetDir` changes `CardTargetDirs` for external-test units only, reaching webster's gate scope and the batcher's read set with no code change there; `brokenFabricVocabularyRule` keeps its behavior.

**Commit:** 1: Map an external test package's unit to its package directory (closes #571, #577, #580, #587)
