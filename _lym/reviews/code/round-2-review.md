---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/testcover/testselect_test.go:282
    summary: no row pins that a linked package's own test files are not read for go:embed patterns, the other half of the embed scope
    fix: In TestSelectorSelect add one row, named LinkedTestFileEmbedIsNoSource, after the EmbedPackageDoesNotParse row. Give it committed set to a/assets.go "package a\n" plus a/a_test.go "package a\n\nimport \"embed\"\n\n//go:embed static/page.txt\nvar page embed.FS\n", files set to pageChanged, inert set to inertText, and no wantMiss and no want. The row must replace a/assets.go, because its "//go:embed static" directive would otherwise name page.txt and miss. Against the current code the row selects nothing and passes. It fails if embedPatterns is called with withTests true for a linked package.
---

### [LOW] No row pins that a linked package's test files are not embed sources

**Location:** internal/testcover/testselect_test.go:282 (`TestSelectorSelect`); the behavior is in internal/testcover/testselect.go:441 (`embedPatterns`) and :250 (`linkedChanges` passing `ownPackage` false).

**Issue:** The focus asks whether the importing package's own `_test.go` `go:embed` directives are read while a linked package reads non-test Go files only.
The code does this correctly.
`ownFilesMiss` calls `otherFileMiss(dir, rel, true)`, and `linkedChanges` calls it with false.
`embedPatterns` keys its cache by `embedScope{dir, withTests}` and skips `_test.go` files unless `withTests` is set.
The own-package half is pinned by `OwnTestFileEmbedsInertFile`, which fails if the importing package's test files were skipped.
The linked half has no row: `a/a_test.go` in the fixture holds no `go:embed` line.
A change that read linked test files would only widen, so it is safe, but it would silently drop the narrowing for an inert file a linked test embeds.
The decision record's D5 says the scope reads the patterns "from the recorded Go files", so the linked half is part of the specified behavior.

## Rejected

- `AddedImportOutsideLinkedSet` (`example.com/x`), `AddedImportOfUnlinkedModulePackage` (`m/d`) and `AddedImportOfDotlessNonStandardPath` (`foo/bar`) reach the same `importAllowed` branch.
  Each rejects a different wrong implementation (a dot heuristic, a module-prefix heuristic, a stdlib-by-dot heuristic), so the rows are not redundant enough to file.
- Embed shapes: the quoted, backquoted, `all:`-prefixed, two-pattern and glob rows each fail if `parseEmbedPatterns` or `embedded` broke, because the `committed` override puts the directive in the file that a no-parse implementation would not match.
  The malformed row (`"static` without a closing quote) reaches `MissParse` through `parseEmbedPatterns`.
  Not a defect.
- `EditInsideMethodBody` edits `Box.Size`'s body and expects `TestSize`, which pins the `Type.Method` key against the map's `Funcs`.
  Not a defect.
- D5 case list: marker `BuiltAt` and `Version` mismatch, commit-side parse failure, absent probe file, removed const, var, type and init, method named `max`, own-file add and remove and non-Go change are all present as rows.
  Not a defect.
- Deleted-file imports: `removedImports` makes a deleted file's non-standard-library import a miss and leaves a deleted file's standard-library import alone.
  This matches D4's deleted-file rule and D5's import rule, and `DeletedFile` and `DeletedFileBlankImportsModulePackage` pin both sides.
  Not a defect.
- Dotless non-standard-library path: `standardLibrary` decides by a directory under `GoRoot/src`, not by a dot, so `foo/bar` misses.
  Not a defect.
