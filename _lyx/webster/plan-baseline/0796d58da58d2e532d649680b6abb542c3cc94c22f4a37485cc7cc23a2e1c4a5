# Card 5 — go-only-checks

**Create:**
- `internal/planparser#Plan.GoToolchain`

**Edit:**
- `internal/planparser#checkHandleMalformed`
- `internal/planparser#checkVerifyPackageMissing`
- `internal/planparser#checkVerifyNestedModule`
- `internal/planparser_test#TestValidate_VerifyNestedModule`
- `internal/planparser#RefScope`
- `internal/planparser#targetDir`
- `internal/planparser#CheckFabricVocabularyName`
- `internal/planparser#brokenFabricVocabularyRule` -> `func brokenFabricVocabularyRule(unit, member, last string, goToolchain bool) string`
- `internal/planparser_test#TestCheckFabricVocabularyName`
- `internal/planglyph#planGatePass`
- `internal/planglyph#LaterDeleteReferences`
- `internal/planparser#TestCheckHandleMalformed_FileUnitRule`
- `internal/planparser_test#TestValidate_VerifyPackageMissing`
- `internal/planparser_test#TestRefCovers`
- `internal/planglyph#TestGlyphChain_CallerCoverage`

**Uses:**
- `internal/planglyph#writeLanguagePlan`
- `internal/planglyph#writeFixtureFiles`

**Intent:** Run the checks that read Go source or Go toolchain semantics only when the plan's language is `go` (D5).

`Plan.GoToolchain` reports whether the plan's `language:` is `go` or absent; it is false under `csharp`, `none` and an unrecognized value.
Gated on it:
- `planGatePass` runs `callerCoverageFindings` and `laterImportFindings` only under `go`, so a `csharp` plan gets no `caller-uncovered` and no import-order `delete-before-reference`;
- `LaterDeleteReferences` returns no findings and opens no repository unless the plan is `go`, which covers the scan-only pass `ValidateDispatch` and record-batch run;
- `checkVerifyPackageMissing` returns nothing unless the plan is `go`, so a C# card's Verify package arguments are not checked, those of `lyx gate test` included;
- `checkVerifyNestedModule` returns nothing unless the plan is `go`, since a nested module is a Go module boundary, the same Go toolchain semantics the package check reads;
- `checkHandleMalformed`'s `.go` file-unit rule fires only under `go`;
- `RefScope` and `targetDir` map a glyph's unit through `goPackageDir`, the external-test unit mapping, only under `go`, and `brokenFabricVocabularyRule` maps a handle's unit through it only under `go`, `CheckFabricVocabularyName` passing whether the plan is `go`.
Build-constraint partitioning keeps its own key (card 8).

What this lets through: a `csharp` plan gets no caller-coverage, import-order or verify-package finding, so a C# card's Verify package arguments go unchecked; every check resting on quarry's answers (resolve, Create, done, scope, drift, delta) still runs.

Tests:
- `TestGlyphChain_CallerCoverage` gains a `language` field and rows under `csharp`, each asserting only the gated checks' findings, so `plan-language-mismatch` from card 6 does not enter them.
  A pair of rows writes, with `writeFixtureFiles`, a nested package `nested/inner/inner.go` (`package inner`, `func Help() {}`) and `nested/user/user.go` importing `example.com/glyphchain/nested/inner`: card 1 deletes the slashed package self glyph `nested/inner#`, which the Go alphabet parses under `csharp` too, and card 2 edits `nested/user/user.go`.
  The `go` row reports the blocking `delete-before-reference`; the `csharp` row reports none, which only the `GoToolchain` gate makes so.
  A Delete of `Glyphchain.Sharp#Renderer.Title` beside a Go file of the fixture copy whose code references an identifier `Title` reports no `caller-uncovered` under `csharp`.
- `TestValidate_VerifyPackageMissing` gains a `csharp` row whose Verify `lyx gate test --wait --tier 0 ./nowhere` draws no `verify-package-missing`, beside the same Verify under `go`, which draws one.
- `TestCheckFabricVocabularyName` gains a `csharp` copy of its row for an owner package's external test package, `plan:internal/fabricengine_test#Test<Side>Thing`, whose unit ends `_test` and whose directory is used unchanged, so it is one finding where the `go` row draws none.
- `TestCheckHandleMalformed_FileUnitRule` gains a `csharp` row whose Create handle `plan:internal/x/file.go#Foo` draws no file-unit finding.
- `TestValidate_VerifyNestedModule` gains a `csharp` copy of its first firing row, which draws no `verify-nested-module`.
- `TestRefCovers` gains `csharp` rows: the package self glyph `x/pkg#` does not cover `x/pkg_test#`, which it covers under `go`; and the file cover `src/Foo.cs` does not cover the C# member `Glyphchain.Sharp#Renderer.Title`, which maps to no scope.

**ImpactSummary:** adds one exported plan method and gates the existing Go-toolchain checks on it; every `go` and absent-language plan keeps its findings, and `none` already skipped all of them.
