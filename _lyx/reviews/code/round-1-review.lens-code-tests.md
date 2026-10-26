```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour, the test that pins it.

**List**
- AT_CLKTCK lookup, absent-entry fallback, zero-rate fallback, nil vector: `TestClockTicksFromAuxv` (`internal/reedengine/proctree_test.go:18-38`), four rows, one per branch. Pinned.
- `paneProcessAge` using the auxv rate (the `unix.Auxv()` call and the `_, _` error swallow): the existing `paneProcessAge` test at `proctree_test.go:79-89` only checks that the age falls in [0, 1h) and ok is true. It does not catch a wrong rate (100 vs. 250 would stay inside the window), and it cannot fail if `unix.Auxv` errors, because the fallback is 100. No test pins this wiring; see F1.
- Key value 17 for AT_CLKTCK: the `DeclaredRate` row uses a literal `17` against `auxvClockTicksKey`. The row would fail if the constant changed. Pinned.
- No `import "C"` in the module's real tree: `TestNoCgo_NoFileOfTheModuleImportsC` (`cmd/lyx/nocgo_test.go:42-50`). It carries `//lyx:guard`, the `tier0` tag and `RequireFloor`. Pinned.
- Fixture file importing `"C"` fails, grouped import, and `_test` file: `TestNoCgo_FixtureTrees` rows `CgoImport`, `CgoImportInGroup`, `CgoImportInTestFile`. Pinned.
- Mention in a comment or a string passes: rows `MentionInComment` and `MentionInString`. Pinned.
- The guard's walk covers nested directories and `_test` files with `scankit.All`: the real-tree test and the `CgoImportInTestFile` row cover it. Covered.
- The failure message's way forward ("dependency in the module cache"): no assertion. This is stencil-like static prose, so it is exempt.

**Findings**
- F1, LOW, class `design`: the wiring of the auxv rate into `paneProcessAge` has no test that pins it. `proctree_test.go:79-89` only checks that the age is in [0, 1h), so a swapped rate or a swallowed `Auxv` error would not fail it. The acceptance criterion says `paneProcessAge` returns the same age as before. A hermetic fix would take an injectable tick source, which the diff does not add. The uncovered behaviour is the `unix.Auxv()` call at `proctree_linux.go:35-37`.
- No duplicated or per-helper tests, and no review-fix tests. `TestNoCgo_FixtureTrees` pins behaviour the real-tree guard cannot (reject versus pass). `scankit_test.go` covers the walker, not the `"C"` import check, so I found no existing test the new ones duplicate.

**Tool calls**
1. A grep and sed over `paneProcessAge` tests, `scankit` users and `scankit.Options`: the manifest lacked the existing `paneProcessAge` test body, the other guard tests and the `Options`/`skipDir` definitions.
