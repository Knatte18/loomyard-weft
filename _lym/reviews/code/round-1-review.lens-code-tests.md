```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it, or none. All pinning is in `TestSelectorSelect` (`internal/testcover/testselect_test.go`), by row name.

**Pinned**
- **D4, edit inside a function body, parameter-type change, deleted file, one of two `init` edited:** `EditInsideFunctionBody`, `ChangedParameterTypes`, `DeletedFile`, `EditToOneOfTwoInits`.
- **No selection when nothing changed:** `NoChange`.
- **Spawn-marked test selected on any change:** the `TestSpawn` entry in each `want`.
- **Comment and formatting changes, reordered imports:** `CommentAndFormatting`, `ReorderedImports`.
- **Allowed additions:** `AddedConst`, `AddedVarWithoutInitializer`, `AddedTypeWithoutMethods`, `AddedFunctionWithStandardLibraryImport`, `AddedImportOfLinkedPackage`.
- **Non-body misses:** `ChangedConst`, `ChangedStructField`, `AddedVarWithInitializer`, `ConstAddedToIotaGroup`, `RemovedImport`, `AddedImportOutsideLinkedSet`, `AddedImportOfUnlinkedModulePackage`.
- **Directive:** `AddedDirective`, `ChangedDirective`.
- **Method set:** `ReceiverValueToPointer`, `AddedMethod`.
- **Added `init`:** `AddedInitInNewFile`.
- **Predeclared name (function):** `AddedFunctionNamedMax`.
- **Parse failure, working-tree side:** `FileDoesNotParse`.
- **Map misses:** `NoMap`, `MapUnreadable`, `BuildFailure`, `NoMarker`, `MarkerDiffersFromMap` (`Commit` field only), `TestNamesFunctionOutsideTable`, `CommitUnreadable` (error branch of `commitMiss`).
- **Own test file edited:** `OwnTestFileEdited`.
- **Non-Go files:** `EmbeddedFileInert` (directory pattern), `NonGoFileNotInert`, `UntrackedNonGoFile`, `InertFileNoGoEmbedNames`.
- **Changes that select nothing:** `LinkedPackageTestFile`, `NestedPackageFile`.

**Behaviours with no row** (`internal/testcover/testselect.go` and `declcompare.go`)

1. **LOW, consistency.** `testselect.go:170` — a marker with a different `Version` is not pinned, though the criterion is "lacks D2's format marker, or a mismatched one". A marker whose `BuiltAt` differs (`testselect.go:172`) is not pinned either. `MarkerDiffersFromMap` edits only `Commit`, so dropping the `BuiltAt` clause would pass. Fix: add `Version` and `BuiltAt` rows.
2. **MEDIUM, design.** `testselect.go:259` and `declcompare.go:157` — no row edits a method body, so `TestSize` is never selected anywhere. The `Box.Size` key naming for methods (the D4 "including methods" focus) is therefore unpinned, and so are pointer-receiver naming and closures inside a function. Fix: add a row editing the body of `Box.Size`, wanting `TestSize` and `TestSpawn`.
3. **MEDIUM, design.** `declcompare.go:113-122` — D4 says init functions "count as changed together when ... their count differs". Only an edit to one init is tested. Removing an init, or an init-count decrease, has no row. Fix: add a row that deletes one `init` and wants `TestInit`.
4. **MEDIUM, design.** `declcompare.go:85-89` — a removed const, var or type, and a changed or removed var or type, have no row. Only a changed const and a changed struct field are tested. The package-clause change (`declcompare.go:74`) has no row either.
5. **MEDIUM, design.** `declcompare.go:169, 201` — an implicit-value const group without `iota` (the "implicit value" half of D5) is not tested. Only the `iota` group is.
6. **LOW, design.** `declcompare.go:77` — a removed `//go:` line, and one inside a function body, have no row. Added and changed lines are tested.
7. **LOW, design.** `declcompare.go:95` — a predeclared name added as a var, const or type, and a method named `max` that must be no miss (`Recv == nil` guard at `declcompare.go:108`), have no row. Only a function is tested.
8. **MEDIUM, design.** `testselect.go:276` — a method set change has two untested cases: a removed method, and a changed signature with the same receiver. Moving a method between files being no miss is a deliberate design decision in the change description, with no row.
9. **MEDIUM, design.** `testselect.go:361-373` and `209-222` — a parse failure at the map's commit has no row; only the working-tree side is tested. The `!found` branch of `commitMiss` (the probe file absent at the commit) has no row. Neither has the `compareGoFile` commit-read failure.
10. **MEDIUM, design.** `testselect.go:415-494` — the go:embed handling has no direct row: `parseEmbedPatterns` quoted and backquoted patterns, the `all:` prefix and multiple patterns; glob patterns in `embedded`; a malformed embed line, which gives `MissParse`; and a Go file that does not parse in `embedPatterns`. Only the plain directory pattern is tested. The D5 criterion is covered for that one shape.
11. **LOW, design.** `testselect.go:188` — the package's own non-Go file changes, an added or removed own Go source file, and own `OtherFiles` are not covered. The fixture's package `b` has only Go files. The change description's own-package non-Go design decision is unpinned.
12. **LOW, design.** `testselect.go:276` — a removed non-Go file recorded in the map (present on one side only) has no row.
13. **LOW, design.** `testselect.go:132`, `335`, `359` — no row calls `Select` twice, so the caches are unpinned. That covers the cache keyed by commit and file shared across importing packages with different `Linked` records, and the sorting and `Compact` of duplicate test names. The error return for an unreadable working file also has no row.
14. **LOW, consistency.** `packagedirs.go` — `NestedPackageDirs` is exported but has no direct test in `internal/testcover`. The grep found no `NestedPackageDirs` or `readPackageTree` test in `internal/gatecli` either. The only pin is `NestedPackageFile`, a plain nested package. Nothing pins a nested `go.mod`, or `testdata`, dot and underscore directories. Fix: add a `NestedPackageDirs` table, or confirm that existing `gatecli` testmap integration tests cover those shapes.
15. **LOW, design.** The `wantMiss` check asserts only a non-empty `Detail`, not its content. This is acceptable, but the `MissOwnFiles`, `MissNonGoFile` and `MissInit` details are unchecked.

Tool calls:
1. `grep` over `internal/gatecli/*_test.go` for tests of the nested-directory shapes, because the manifest holds no test of `NestedPackageDirs` or `readPackageTree`.
