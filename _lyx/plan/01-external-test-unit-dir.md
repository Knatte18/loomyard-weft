# Card 1 — external-test-unit-dir

**Create:**
- `plan:internal/planparser#externalTestUnitDir` -> `func externalTestUnitDir(unit string) string`

**Edit:**
- `internal/planparser#unitDir`
- `internal/planparser#CardTargetDirs`
- `internal/planparser#brokenFabricVocabularyRule`
- `internal/planparser_test#TestCardTargetDirs`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine#TestRenderBatchGate`
- `internal/batcher_test#TestPeakContext`
- `internal/planparser/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-glyph-conversion-chokepoint.md`

**Intent:** Map a Go external-test glyph unit to its package directory (D1), and record that mapping as the one sanctioned exception to the glyph chokepoint (D2).

Quarry names an external test package's unit `<dir>_test`, a pseudo-path with no directory, and `Glyph.UnitPath` returns it unchanged, so today a card targeting `internal/x_test#TestY` yields the gate argument `./internal/x_test`.

- `externalTestUnitDir` lives in `targetdirs.go`.
  It takes a Go unit string, either a glyph's `UnitPath` result or a handle's package half, and when the unit's last path segment ends in `_test` and has no file extension, returns the unit with that `_test` suffix removed; any other unit is returned unchanged.
  It reads no disk.
- `unitDir` applies it to a unit that came from a glyph or handle (`fromGlyph` true) in its no-extension (package) branch, so every glyph- or handle-derived package unit maps through it.
  A path ref (`fromGlyph` false) and a file unit (a unit with an extension, such as `internal/x/x_test.go`) are untouched, and `RefFile` is unchanged because a package unit names no file.
- `CardTargetDirs`' doc comment gains the sentence that an external-test unit maps to its package directory, so `internal/x#Run` and `internal/x_test#TestRun` on one card yield one `internal/x` entry.
- `brokenFabricVocabularyRule` replaces its `strings.TrimSuffix(unit, "_test")` with `externalTestUnitDir(unit)`; the comment above it stays.
  Its result is unchanged for every handle unit, since a handle's unit is a package directory: `TestCheckFabricVocabularyName`'s existing `internal/configsync_test` and `internal/fabricengine_test` rows pin that behaviour and stay green unedited.

Both production readers of `CardTargetDirs` change with it, with no code change of their own: webster's `gateScopeOf` (card and batch gate steps) and `internal/batcher`'s read-set estimate, whose `package:` and `tests:` keys for `internal/x_test` become `internal/x` and merge with an existing entry.

Bound: the mapping is disk-free and unconditional, so a real Go package directory whose own name ends in `_test` would map to its sibling; this repository has none.

Tests, all rows in existing tables:

- `TestCardTargetDirs` gains rows: a member glyph `internal/a_test#TestRun` in an `Edit` group maps to `{Dir: "internal/a", NamesGo: true}`;
  `internal/a#Run` beside `internal/a_test#TestRun` on one card yields the single entry `{Dir: "internal/a", NamesGo: true}`;
  a `Create` handle `plan:internal/a_test#TestNew` maps to `internal/a`;
  a plain directory path ref `internal/a_test` stays `{Dir: "internal/a_test"}`, since path refs are untouched.
- `TestCardGateCommand` gains a subtest row: a card targeting `internal/a#Run` and `internal/a_test#TestRun` renders `lyx gate test ./internal/a` once, then the two lints.
- `TestRenderBatchGate` gains a row: a two-card batch, card 1 targeting `internal/a_test#TestRun` and card 2 `internal/a/a.go#`, names the root module's packages as `./internal/a` alone and the batch gate as `lyx gate test --tags integration ./internal/a`.
- `TestPeakContext` gains a table row: one card editing `internal/a/a.go` and `internal/a_test#TestRun` peaks at `startup + 2*3 + 2*4 + 50 + 7 + 7`, which counts the `internal/a` package and test entries once; before the change the `internal/a_test` entries add a second `7 + 7`.

Docs in this commit:

- `PATTERN.md`'s `PATTERN-glyph-conversion-chokepoint` line becomes, verbatim:

```markdown
- `PATTERN-glyph-conversion-chokepoint` — Converting glyphs and paths: only `glyph`'s `Self`, `UnitPath`, `Parse` and `String`, plus `planparser`'s one external-test unit mapping. — [background](pattern/PATTERN-glyph-conversion-chokepoint.md)
```

- `pattern/PATTERN-glyph-conversion-chokepoint.md` gains a bullet naming `planparser`'s `externalTestUnitDir` as the single sanctioned adjustment of a Go unit string, for the external-test pseudo-unit only: quarry's `docs/glyph.md` "Unit corner cases" documents `<dir>_test` as a pseudo-path with no directory, so a caller must map it, and `CardTargetDirs` and the fabric-vocabulary name check are its callers.
- `internal/planparser/doc.go`'s "Glyph Conversion Chokepoint Invariant" section names the same exception in one sentence.

`lyx loom lint-pattern` passes with the new line.

**ImpactSummary:** Changes the directory every glyph- or handle-derived external-test unit maps to inside planparser, which reaches webster's card and batch gates and the batcher's estimate through `CardTargetDirs` with no code change there; path refs and file units are unaffected.

**Commit:** `1: external-test glyph units map to their package directory`
