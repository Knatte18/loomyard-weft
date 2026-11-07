---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/fabricengine/midmerge_integration_test.go:41
    summary: TestMidMerge_FabricParkedWithConflicts_Parked repeats the clashLayer steps by hand and was not moved onto it
    fix: In TestMidMerge_FabricParkedWithConflicts_Parked, replace the three lines from newMergePairFixture(t, ".") through branchAtCurrentHEAD(t, h.PrimeRecords(), "feature-weft") with h, f, _, _, _, _ := newMergeLayerFixture(t, &clashLayer). Keep the wantStartSHA and wantSourceSHA reads below it unchanged.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: consistency
    location: internal/fabricengine/livestate_verbs_test.go:664
    summary: The Cleanup Expect comment still says Arrange moves the prime pair off f.OriginalBranch, but that step now runs in Build
    fix: In the comment above `primary := fabricengine.RecordsBranchName(f.OriginalBranch)` in cleanupCase, change "Arrange moves the prime pair off f.OriginalBranch," to "Build moves the prime pair off f.OriginalBranch,".
    origin: lens:code-affected-set
---

### [LOW] midmerge test repeats the clashLayer steps and was not moved

**Location:** `internal/fabricengine/midmerge_integration_test.go:41`
**Issue:** `TestMidMerge_FabricParkedWithConflicts_Parked` calls `newMergePairFixture(t, ".")`, then `setupConflictingDivergence(... "feature", "clash.txt")`, then `branchAtCurrentHEAD(... "feature-weft")`.
These are the steps `clashLayer` builds, in the same order.
The layer exists because at least three sites share them, and this site is one of them.
The commit message says only sites that do not repeat the commits keep their own steps, so this site contradicts it.
The other similar sites differ in the weft step, a seed commit, a decoy branch or a non-prime pair, so they are rightly left alone.

### [LOW] Stale Arrange comment in the Cleanup expectation

**Location:** `internal/fabricengine/livestate_verbs_test.go:664`
**Issue:** The comment in the Cleanup `Expect` reads "Arrange moves the prime pair off f.OriginalBranch".
The move to `verb-cleanup-alt` is now in `Build`, and the commit renamed the neighbouring comments, so this one disagrees with the code and with them.

## Rejected

- Lens code-correctness, LOW: `mustGit` inside the Cleanup `Build` panics on the template-build goroutine.
  The old `Arrange` called the same `mustGit` and panicked on the test goroutine, which also ends the test binary, so the failure mode did not degrade.
- Lens code-tests, NIT: the CloneHub{Reset} column uses `CopyHub` and `Arrange` instead of `copyVerbHub`.
  No case in that column has a `Build`, and D7 leaves those cases whole, so nothing is wrong today.
- `cleanMergeableLayer` has only two source sites.
  `newCleanMergeablePair` is called by two tests, so the start state is built three times, which meets D7's threshold.

## Judged against the focus directive

- Every registered `Build` calls only `AddPair`, `os.RemoveAll` inside the hub and `mustGit` on hub paths.
  None calls `os.Setenv`, `os.MkdirTemp`, global git config, a seam setter or a once-only registration, and none keeps the TB's `Context()`.
- Each `Build` moved out of an `Arrange` leaves the fixture value the same.
  Cleanup's `OriginalBranch` now reads `BoardDir()`, which is the Bolt checkout (`BoardDirName` equals `BoltDirName`), and `primaryWeftBranch` reads the same branch.
- Add, Pull, Reconcile, the hostile cases and the CloneHub{Reset} column keep their `Arrange` whole, as D7 requires.
  Pull pushes into the code bare through a clone outside the hub, and Add plants a launcher blocker.
- Each layer reproduces the replaced commit steps exactly.
  The merge tests install no process-global seam, so the copy-before-install rule holds.
- `weftguards_integration_test.go` and `mergein_recovery_integration_test.go` drop no assertion.
- The diff touches only `internal/fabricengine`, and per-row slugs chosen by a test stay in the test.
- The commit messages match the diff: six layers moved, and the five verbs listed moved into `Build`.
