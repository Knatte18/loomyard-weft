# Card 2 — test-file-ways-forward

**Edit:**
- `internal/planglyph#coverageVerbAndWay` -> `func coverageVerbAndWay(subject coverageSubject, files []string) (verb, way string)`
- `internal/planglyph#uncoveredFinding`
- `internal/planglyph#unresolvedMethodFinding`
- `internal/planglyph#planGatePass`
- `internal/planglyph#TestGlyphChain_CallerCoverage`
- `internal/planglyph#TestGlyphChain_RedundantFile`
- `contracts/specs/loom-plan-spec.md`

**Intent:** Make the `caller-uncovered` and `redundant-file-target` ways forward stop offering a `_test.go` file, which card 1's `edit-test-file` refuses under `**Edit:**` (the record's parent decision "Gate ways forward and Plan-Review rubric follow the test-file refusal", items 1 and 2).

`coverageVerbAndWay` takes the files the finding names.
When any of them ends in `_test.go`, its way names only the member glyph whose body holds the reference, on that card (or on that card or an earlier one for a delete), and says a `_test.go` file is refused under `**Edit:**` by `edit-test-file`; otherwise its way is unchanged.
`uncoveredFinding` passes its one file, and `unresolvedMethodFinding` passes its `files`, so a method's informational finding that names a test file among others names only the member glyph, which covers every file it names.

In `planGatePass`, the `redundant-file-target` detail for a file ending in `_test.go` drops "or keep the file when the card changes the whole file" and says to keep the member glyphs and drop the file, naming `edit-test-file` as the reason; a non-test file keeps today's wording.

Both carve-outs key on the `_test.go` suffix alone, so a `_test.go` file under a directory the go tool ignores, which `edit-test-file` exempts, also gets the member-only way; the member glyph still covers it, so the way stays admissible.

`contracts/specs/loom-plan-spec.md` rows 33 (`redundant-file-target`) and 35 (`caller-uncovered`) state the same carve-out in their way-forward sentences.

Tests:
- `TestGlyphChain_CallerCoverage`'s finding loop asserts, for every `caller-uncovered` finding, that its detail carries "list the file" exactly when none of the coverage files it names (`coverageFileOf`) ends in `_test.go`.
  Its existing rows already report blocking findings on `callees/callees_external_test.go` and on `callees/local.go`, so both branches are reached.
- `TestGlyphChain_RedundantFile` gains a `keepFile` field per row, true on its existing rows that report a finding, and its loop asserts that each `redundant-file-target` detail carries "keep the file when the card changes the whole file" exactly when `keepFile` is set.
  A new row "test file beside a member declared in it" targets `shapes/shapes_test.go` and `shapes#testHelper`, which the fixture declares there, and wants one blocking `redundant-file-target` on `1-card1` with `keepFile` false.
  It runs through the package's `ValidateFormat`, which does not append `edit-test-file`, so the row's finding set is the redundancy finding alone.

**ImpactSummary:** Two way-forward texts in `internal/planglyph` change for `_test.go` files only; `coverageVerbAndWay`'s two callers are in this card, and no test outside the two listed asserts either wording.

**Commit:** `2: Gate ways forward name a test file's members, never the file`
