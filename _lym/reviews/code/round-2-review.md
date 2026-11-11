---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/gatecli/testmap_integration_test.go:920
    summary: nothing pins that a package skipped on a linked file dirty after its tests still counts against --max
    fix: In TestGateTestMapLinkedPackages, subtest ALinkedFileDirtiedUnderTheTestsWritesNoMapAndIsSeenAfreshByALaterPackage, call callMax with a cap of 1 over "dirtier", "follower" and assert the outcome's Unbuilt is 1 and follower is not tried for a build. Alternatively add the cap assertion as a row to SkippedPackageCountsAgainstMax. The after-tests dirty branch in mapPackage returns false separately from the changed-files branch, so only a test of this branch pins it.
  - id: F2
    severity: LOW
    class: design
    location: internal/gatecli/testmaptree.go:156
    summary: the added-file and removed-file branches of changedFiles have no test, for a linked package or for the own package
    fix: Extend ALinkedFileChangedUnderTheTestsWritesNoMap with a stand-in variant that commits a new file to the linked package, or one that removes a tracked file, and assert the skip cause names that file as "linked/<file>". The stand-in's appendAndCommit already creates a missing file, so a new test name suffix and testMapLandEnv-style variable can drive the added branch.
---

### [LOW] An after-tests dirty skip is not pinned as counting against --max

**Location:** internal/gatecli/testmap_integration_test.go:920
**Issue:** The focus asks whether the before-build skip leaves the count untouched while the after-tests skip still counts.
The code is right: `mapPackage` returns true only for the before-build skip, and `mapPackages` then decrements `built`.
The after-tests dirty skip and the files-changed skip return false, so both count.
The before-build side is pinned by `ADirtyLinkedPackageLeavesTheMapAsItWasAndCountsNothingAgainstTheCap`, a cap of 1 that still builds `linked/inner`.
The files-changed side is pinned by `SkippedPackageCountsAgainstMax`.
The after-tests dirty branch has its own `return false, nil`, and the one subtest that reaches it runs with a cap of 10, so a change of that return to true passes every test.

### [LOW] changedFiles' added-file and removed-file branches are untested

**Location:** internal/gatecli/testmaptree.go:156
**Issue:** The decision requires the changed-while-tests-ran check to cover every linked package's files, and the doc says a file added or removed since counts as changed.
`ALinkedFileChangedUnderTheTestsWritesNoMap` and the own-package `moved` row cover only a content change to an existing file.
The loop over `after.hashes` for a file absent before, and the `after.hashes[rel] != before.hashes[rel]` comparison for a removed file, are behavior the diff adds with no test.

## Focus answers

- `mapPackages` and `mapPackage` skip accounting: a package skipped on the before-build dirty read of a linked file leaves `--max` untouched, and the after-tests skips count.
  The `testMapCmd` Long text, the `mapPackages` doc comment, the `mapPackage` doc comment and the testmap section of `doc.go` all say so, and `doc.go` also names the `go list` the early skip costs.
  Not a defect, apart from F1's missing pin.
- `TestGateTestMapLinkedPackages` and `TestGateTestMap`:
  breaking the linked HEAD-moved check fails `ALinkedFileChangedUnderTheTestsWritesNoMap`;
  dropping `Commit` or `BuiltAt` from a failed-build map's marker fails the broken-map marker assertions and the go-list and spawn-scan failure subtests;
  a before-build read that reused the walk's list fails the dirtier/follower subtest, since the scratch file does not exist when the call starts.
  Not a defect.
- `nestedPackageDirs`: the nested go.mod branch (`linked/modnest`), the nested package branch (`linked/inner`) and the testdata, underscore and dot directory exclusions (`linked/testdata`, `linked/_hidden`, `linked/.dotdir`) each have a row in `wantLinked`, which asserts both what is owned and what is left out.
  The doc comment states the build-constraint bound.
  Not a defect.
