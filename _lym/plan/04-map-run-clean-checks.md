# Card 4 — map-run-clean-checks

**Edit:**
- `internal/gatecli#testMapSkip`
- `internal/gatecli#testMapRun`
- `internal/gatecli#testMapRun.skip`
- `internal/gatecli#runTestMap`
- `internal/gatecli#testMapRun.mapPackages`
- `internal/gatecli#testMapRun.mapPackage`
- `internal/gatecli#testMapRun.measureTests`
- `internal/gatecli#testMapRun.endJob`
- `internal/gatecli#testMapRun.write` -> `func (r *testMapRun) write(pkg string, m testcover.Map, trees map[string]packageTree) error`
- `internal/gatecli#testMapRun.writeFailed` -> `func (r *testMapRun) writeFailed(pkg string, m testcover.Map, failure error, trees map[string]packageTree) error`
- `internal/gatecli/doc.go`
- `internal/gatecli#TestGateTestMap`
- `internal/gatecli#TestGateTestMapLinkedPackages`
- `internal/gatecli#testMapOutcome`
- `docs/overview.md`
- `pattern/PATTERN-test-speed.md`

**Delete:**
- `internal/gatecli#testMapRun.changedFiles`
- `internal/gatecli#changedName`

**Create:**
- `internal/gatecli#testMapRun.checkClean`
- `internal/gatecli#testMapRun.dirtyModuleFile`

**Uses:**
- `internal/gatecli#testMapRun.dirtyOwnedFile`
- `internal/gatecli#testMapRun.readHead`
- `internal/gatecli#testMapRun.readPackageTree`

**Intent:** Write a package's map only when HEAD is still the commit the run read at its start and no file the package or a linked package owns is modified or untracked, checked once the build ended and again before the write; list every package left unmeasured with its reason and exit non-zero (D6).

- `testMapRun.checkClean(trees)` reads HEAD through `readHead` and the worktree status through `dirtyOwnedFile`, and returns the unmeasured reason, `""` when the package may go on.
  The reason is `HEAD moved from <start> to <now>; way forward: re-run the same command once HEAD stays put` once HEAD differs from the run's start, else `modified or untracked file <path>; way forward: commit or revert it, then re-run the same command` for the first dirty file a package of `trees` owns, a package owning the files under its directory down to the next package directory or nested module.
  A nil `trees`, for a build whose listing of its linked packages failed, checks HEAD and `testMapRun.dirtyModuleFile()`, the first modified or untracked file under the module root, with the same reasons.
- Every map write goes through a check: `testMapRun.write` and `testMapRun.writeFailed` take the trees to check and run `checkClean` over them first, writing nothing when it gives a reason.
  So each write path is guarded in one place:
  - a package with no mapped test, and one whose mapped tests are all spawn-marked, both of which have no build, are checked once before their write, over the package's own tree;
  - a test or spawn scan failure before any build is checked over the package's own tree before its failure is written;
  - a failed coverage build is checked over the linked trees, or over the module when the listing failed, before its failure is written;
  - a failure while the tests run, such as a profile that cannot be reduced, is checked over the linked trees before its failure is written.
    This path shares `checkClean`'s call inside `writeFailed` with the failed-build path, which `AFailedBuildOverADirtyLinkedFileWritesNoMap` reaches, and no separate input reaches it.
- `testMapRun.mapPackage` also runs `checkClean` over the linked trees once the coverage build ended and compiled, before the tests run, so a package dirty at that point runs no test.
- `testMapRun.endJob` runs `checkClean` over the linked trees after the tests ran, in place of its first-dirty-file read and its HEAD comparison through `changedFiles`; its write then goes through `write`'s own check.
  `changedFiles` and `changedName` go, and `testmaptree.go`'s file comment drops "which of them changed between two commits".
- A failed check writes no map: the package joins `testMapRun`'s `unmeasured` list, printed under the envelope key `unmeasured`.
  `testMapSkip` renames its `Cause` field to `Reason` with the JSON key `reason`, and `testMapRun.skip` appends to `unmeasured`; `skipped` leaves the envelope.
- A dirty package does not end the call, and the run goes on to the next package.
  A moved HEAD ends it: `testMapRun.mapPackages` lists the current package and every package not yet measured as unmeasured with the HEAD-moved reason, replacing the `queued` count, which goes from `testMapRun` and the envelope.
  It no longer reads HEAD before each package, since every write is checked.
- `runTestMap` returns 1 when `unmeasured` is not empty, beside card 3's causes.

Bound: an edit made before a package's first check, or between its two checks, and reverted before the next check goes unseen, so the map holds coverage of the edited code under the committed hashes; a gitignored file is seen by neither check, so a gitignored Go file the build compiles enters the map under HEAD's hashes.

Tests:
- `testMapOutcome` replaces `Skipped` with `Unmeasured` and drops `Queued`.
- `TestGateTestMap`:
  - The zmoved and amoves subtests want zmoved, or amoves and bhangs, unmeasured with the HEAD-moved reason and exit 1.
  - New subtest `APackageWithNoMappedTestIsCheckedBeforeItsWrite`: an untracked file in `emptied`, whose test file card 1's subtest deleted, leaves emptied's map as it was, lists emptied unmeasured naming the file, and exits 1; the subtest removes the file at its end.
- `TestGateTestMapLinkedPackages`:
  - `ADirtyLinkedPackageLeavesTheMapAsItWasAndCountsNothingAgainstTheCap` modifies the tracked `linked/linked.go` in place of the untracked `notes.txt`, wants caller and follower unmeasured naming `linked/linked.go`, linked/inner built, and exit 1, and restores the file (D6).
  - `ALinkedFileDirtiedUnderTheTestsWritesNoMapAndIsSeenAfreshByALaterPackage` wants both unmeasured naming `linked/scratch.txt`, dirtier through the check before its write and follower through the check after its build.
  - `ALinkedFileChangedUnderAJobDiscardsAllItsRunsAndEndsTheCall` and `ALinkedFileAddedOrRemovedUnderAJobDiscardsIt` want the HEAD-moved reason, every later package unmeasured with it, the maps left and exit 1 (D6).
  - New subtest `AFailedListingThroughADirtyFileWritesNoMap`: `testMapFailListEnv` set and an untracked file under `follower` leave caller's map as it was, list caller unmeasured naming the file and exit 1, where `AGoListFailureKeepsTheMapsEntriesAndRecordsTheFailureWithHead` over a clean tree still records the failure (D6).
  - New subtest `AFailedBuildOverADirtyLinkedFileWritesNoMap`: `testMapFailBuildEnv` naming caller and an untracked `linked/notes.txt` leave caller's map as it was, list caller unmeasured naming the file and exit 1; the subtest removes the file.
  - New subtest `AFailedSpawnScanOverADirtyFileWritesNoMap`: an untracked `garbled/notes.txt` beside the package `ASpawnScanFailureWritesAFailedMapWithTheMarker` committed leaves garbled's map as it was, lists garbled unmeasured naming the file and exits 1; the subtest removes the file.
  - New subtest `APackageWhoseTestsAreAllSpawnMarkedIsCheckedBeforeItsWrite`: a committed package `spawner` whose one tier1 test runs `os.Executable`, mapped once clean, then with an untracked `spawner/notes.txt`, keeps the first map, lists spawner unmeasured naming the file and exits 1; the subtest removes the file.

Docs: `internal/gatecli/doc.go`'s `# Coverage maps` section, `docs/overview.md` and `pattern/PATTERN-test-speed.md`'s coverage-map section state the two checks, the guarded write paths, the unmeasured list, the HEAD-moved end, the exit and the bound above.

**ImpactSummary:** Confined to the map run in gatecli: its status and HEAD checks move to after each build and into each map write, its skip and queued reporting become one unmeasured list, two HEAD-diff helpers go, and the two write helpers take the trees to check; every caller is in the map run itself.
