# Card 5 — seam-marker-parser

**Create:**
- `internal/impactset#SeamTest`
- `internal/impactset#SeamMalformed`
- `internal/impactset#FindSeamTests`
- `internal/impactset#TestFindSeamTests`

**Edit:**
- `internal/impactset#onlyKeepLinesAfter`
- `internal/impactset#TestFindGuardTests`
- `internal/impactset#GuardScanError`
- `PATTERN.md`
- `internal/impactset/doc.go`
- `pattern/PATTERN-test-speed.md`

**Uses:**
- `internal/impactset#guardNamesInFile`
- `internal/impactset#writeTestFile`

**Intent:**
D5's marker and its parser, beside `//lym:guard` in `internal/impactset`, modelled on `guardNamesInFile`.

A test function whose body reflects over a struct's field set carries `//lym:seam <unit>#<Type>`, one line per struct, in the comment block directly above its `func Test…` line.
`//lym:seam`, `//lym:guard` and `//testtiming:keep` lines stack there in any order:
`onlyKeepLinesAfter` now admits `//lym:seam …` lines as well as `//testtiming:keep` lines between a `//lym:guard` marker and its func line, so the guard scan accepts a seam line on either side of the guard marker;
cmd/testtiming already admits any `//lym:` line between a keep line and its func line.

`SeamTest` holds `Name string` (the top-level test), `Unit string` (the test's glyph unit: the package directory, or `<dir>_test` for a file whose package clause ends `_test`), `File string` (worktree-relative, slash-separated) and `Structs []string` (each marker's glyph, in comment order).
`FindSeamTests` parses every `_test.go` file of each told package directory with comments, as `FindGuardTests` does, and returns, per directory, the tests carrying at least one `//lym:seam` line, sorted by name.
A seam line counts when it sits in a top-level `func Test…`'s doc comment and only `//lym:` and `//testtiming:keep` lines follow it before the func line.
A seam line anywhere else is a `*GuardScanError` of kind `GuardMisplaced` naming the file, the line and `//lym:seam`, with "way forward: move the `//lym:seam` line into the comment block directly above the `func Test…` line of the test that reflects"; a seam line whose argument is not one token that holds exactly one `#` with a non-empty unit and member is a `*GuardScanError` of kind `SeamMalformed` naming the file and line, with "way forward: write the marker as `//lym:seam <unit>#<Type>`, one struct per line"; an unparsable file is the `GuardParse` error `guardNamesInFile` already returns.
Whether a marker's glyph names a struct is the scan's business (card 6), not the parser's.

Tests, tier0, in the existing test file of the guard parser:
`TestFindSeamTests`: one marker above a test is found with its unit, file and struct; two markers on one test come back in order; a marker in an external `p_test` package file has unit `p_test`; a test carrying `//lym:guard` then `//lym:seam` and one carrying `//lym:seam` then `//lym:guard`, each with a `//testtiming:keep` line too, are found; a marker above a helper func, or followed by prose before the func line, is `GuardMisplaced` at its line; `//lym:seam` with no argument and `//lym:seam Env` are `SeamMalformed`.
`TestFindGuardTests` gains rows: `//lym:guard` then `//lym:seam p#T` then the func, and `//lym:seam p#T` then `//lym:guard` then the func, both find the guard test.

`PATTERN.md`'s `PATTERN-test-speed` line becomes the declared text:

```
- `PATTERN-test-speed` — Writing or running a test: via `lym gate test` by gate key; one `tierN` tag, the lowest that fits; it pins behavior no test covers; injected clocks, `t.Parallel` by default; a test reflecting over a struct's fields carries `//lym:seam`. (test) — [background](pattern/PATTERN-test-speed.md)
```

`pattern/PATTERN-test-speed.md` gains a "Reflective tests carry `//lym:seam`" section beside its `//lym:guard` section: the marker's form, one line per struct the test's reflection walks, nested structs included; the stacking rule above; that the tier0 scan refuses an unmarked reflective test it sees, a marker naming no struct and a `NilSeams` test missing a struct the `Env` set holds; the bound that reflection through a non-test helper other than `envkit.NilSeams` is not seen; that a struct declared outside the module is marked by its import path, such as `//lym:seam time#Time`, and such a marker, whose unit's first segment names no worktree directory, is not checked against a declaration; and that `lym loom validate-plan` reads the markers for `seam-test-unlisted`.
Its existing sentence that `//lym:guard` counts as directly above when only `//testtiming:keep` lines separate it from the func line is rewritten to admit `//lym:seam` lines as well.
`internal/impactset/doc.go` names `FindSeamTests`, `SeamTest` and the `SeamMalformed` kind, and its sentence that only a `//testtiming:keep` line between a guard marker and its func line leaves the marker in place, and its count of the scan error kinds, are rewritten for the seam line and the fourth kind.
`GuardScanError`'s type doc and its `Kind` field doc ("which of the three failures this is") name the seam failures and the fourth kind.

**ImpactSummary:** the guard scan's directive-line rule widens by the seam line only, so every existing guard test still parses; the seam parser is new surface with no caller until card 6.

**Commit:** `5: Parse the //lym:seam marker beside //lym:guard`
