---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:36
    summary: The change description says a test file's imports get no allowance and a new test file with imports misses, but the selector allows standard-library, own-package and linked-set imports
    fix: In _lym/landing/summary.md, rewrite the "Own test files" paragraph (the sentence listing "an import" among the changes that give MissTestFile) and the design bullet "Imports in a test file get no linked-package allowance" to match compareTestSources and packageChanges. An added import is no miss when it is the package's own import path, in the map's linked set, or in the standard library, and the same holds for a removed import bar the linked set (own or standard library only). Any other added or removed import gives MissTestFile. A new test file importing testing therefore selects its Test functions.
  - id: F2
    severity: MEDIUM
    class: decision
    location: internal/testcover/testselect.go:298
    summary: The import allowance for test files goes beyond the record's rule that any change to a test file outside its Test functions is a miss, and no parent correction records it
    fix: Leave the code in place and defer this finding to the run's parent. The only sound change is recording the allowance as a parent correction with lym loom decision add (or refusing it), which lies outside the fixer's write surface.
---

### [MEDIUM] Change description contradicts the diff on test-file imports

**Location:** `_lym/landing/summary.md:36`
**Issue:** The summary states that a test file's imports get no linked-package allowance and that a new test file with imports misses.
The diff does the opposite after the round-1 fixes.
`packageChanges` (`internal/testcover/testselect.go:298`) accepts an added import that is the package's own, in the map's linked set, or in the standard library.
`packageChanges` accepts a removed import that is the package's own or in the standard library.
The rows `OwnTestFileWithOnlyTestFunctionsAdded`, `ExternalTestFileImportingOwnPackageAdded` and `OwnTestStandardLibraryImportAdded` in `testselect_test.go` pin the allowance.
The summary is the writer's claim and the diff does not bear it out.

### [MEDIUM] Test-file import allowance is an unrecorded departure from the record

**Location:** `internal/testcover/testselect.go:298`
**Issue:** D2 says any change to a test file outside a `Test` function's text, "such as a helper, `TestMain`, an `init` or a package-level declaration", is a miss.
An import change is such a change, yet the selector reads it as no miss in the cases above.
The allowance is sound as code: a test binary already links the package, its linked set and the standard library, and a new test file needs `testing`.
Under the record's text a new test file importing `testing` would miss, so the call changes the record's rule.
Neither the summary nor a `Pattern-Departure:` trailer names it as a correction, and no `Added after Discussion` entry records it.

I checked each focus question against the code and its tests and found no defect:

- `stampEnvironment` and `mappedTierConfigs` are pinned by `TestStampEnvironment` and `TestMappedTierConfigs`.
  A changed tier command, tier env value or hub env variable moves its digest, and a changed tier `parallel` leaves both.
- `compareTestSources` with a new or removed test file importing `testing` is no miss, while an import outside the standard library and the linked set still misses.
  The new `Test` functions are selected, as the rows `OwnTestFileWithOnlyTestFunctionsAdded`, `OwnTestFileImportingTestingRemoved`, `OwnTestFileImportingLinkedPackageRemoved` and `OwnTestImportOutsideLinkedSetAdded` show.
- `TestSelectorSelect` has a miss row for each of the seven stamp fields and asserts the exact detail naming the differing field.
- `mapMiss` leaves a build-failure record beside test entries usable, pinned by `BuildFailureBesideTestEntries`, and misses it with no test entries, pinned by `BuildFailureWithNoTestEntries`.
- The unreadable commit target, the unreadable toolchain in `openSelector` and the test-file package-clause change each have a test.
