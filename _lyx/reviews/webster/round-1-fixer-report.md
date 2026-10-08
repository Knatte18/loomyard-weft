# Webster-Burler round 1 — fixer report

Every finding was validated against the code and the plan, and all nine are fixed, each in its own commit.
Verify after the last fix: `go build ./...`, `go vet` under `integration`, `tmux` and `llm`, `go test ./...` and `go test -tags integration ./...` all pass, and `lyx loom lint-comments --base fd5b0f58e` is clean.

## Fixed

- **F1** (`6bd8239a8`): rewrapped every touched sentence the review named so that it uses semantic line breaks.
  The locations were `doc.go`'s glyph-rejected sentence, `beginbatch.go`, `repo.go`, `planglyph.go`'s ValidateDispatch doc and `handle.go`'s `renameDeclSource` and `cardOwnHandles` docs.
  Each sentence now sits on one line or breaks only at a semicolon, a colon or a comma before an independent clause.
  Where splitting a line touched the next sentence, that sentence was rewrapped too.
- **F2** (`db50c4b0a`): `redundant-package-target` now keys every owner by a `UnitPath` result.
  This covers a member glyph, a Rename pair's old glyph in `renameToSideUnits`, and a handle, whose body is parsed through `glyph.Parse` and converted by `UnitPath` instead of being read raw from `handleUnit`.
  No path is read from `Glyph.Unit` any more.
- **F3** (`c40f30d27`): in the spec, the rewritten handle-checks paragraph and the edited `glyph-malformed` row now hold one sentence per line.
- **F4** (`89379cd93`): `planGatePass` skips a target that is repeated on one card, and `coverageSubjects` skips a repeated `(card, ref)`.
  Regression rows were added to `TestGlyphChain_RedundantFile` and `TestGlyphChain_CallerCoverage`, and both failed before the fix.
- **F5** (`c103e10cd`, `9a1d2b3e5`): `isInterfaceMethod` now recognises a declared method by `func (` or `func(`.
  `renameDeclSource` calls `isInterfaceMethod` instead of duplicating the prefix test.
  `TestGlyphChain_FuncNamedInterfaceMethod` writes `FuncNamed { funcLike() int }` into the fixture copy and pins the arrow, which must give `resign-interface-method`, and the Rename, which must canonicalize.
  Before the fix the arrow gave no finding and the Rename gave `handle-name-failed`.
  The new top-level test needed `internal/planglyph`'s row in `test-budget.yaml` raised from 81 to 82.
- **F6** (`2e2341610`): `TestGlyphChain_CallerCoverage` gained a row that writes callers of `callees.Target` under `testdata/`, `vendor/`, `.hidden/`, `_ignored/` and `callers/testdata/`, and expects no finding.
  Disabling the skip makes the row fail.
- **F7** (`406a2cfe7`): the package overview in `doc.go` now names `planGatePass` after `resolvePass`, and states that `ValidateDispatch` never runs it.
- **F8** (`c42ed52a0`): dropped the `a path beside a package self glyph is clean` row.
  Under a glyph language, `normalizeRefIfPath` turns that raw path into the file self glyph that the row above already flags.
- **F9** (`b30cd5f86`): `TestGlyphChain_UnreferencedHandle` no longer asserts the plan-gate verdict that `TestGlyphChain_CrossCard` pins.
  The test still calls `ValidateFormat` and fails only on an error, because that call canonicalizes the draft handle the bind step matches.
  Removing the call outright makes `BindHandles` report `bind-count-mismatch`.

## Disputed

None.

## Deferred

None.
