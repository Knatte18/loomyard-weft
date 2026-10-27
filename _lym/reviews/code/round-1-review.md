---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/websterengine/state_test.go
    summary: websterengine.LoadState on a worktree with no scratch directory has no test, though an acceptance criterion names it
    fix: Add a case to the LoadState tests (for example a row in TestLoadState_UnusualFiles or a small new test) that points scratchDir at a path that does not exist, calls LoadState, and asserts it returns (nil, nil) and that scratchDir still does not exist afterwards.
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/add_hubreader_integration_test.go:28
    summary: Nothing asserts that ReadOrigin creates no .weft directory and seeds no exclude, which is the behavior change made to ReadOrigin
    fix: In the beforeWire hook of TestAdd_HubReaderBeforeWiringLeavesNoLym, after the ReadOrigin call on the half-built pair, assert with os.Lstat that no .weft directory exists under the pair's weft worktree and that the weft worktree's info/exclude is unchanged from before the read. Alternatively add a ReadOrigin test over a pair whose .weft is absent that asserts the same.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: scope
    location: internal/state/strict_test.go
    summary: No TestReadJSON row pins the bound that any lock error other than not-exist still fails the read
    fix: Add a TestReadJSON row whose lock path cannot be opened for a reason other than a missing directory (for example a directory placed at the lock path, whose parent exists) and assert ReadJSON returns an error and not the unlocked record. A regression to skipping the lock on any lock error then fails.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: scope
    location: internal/orchcli/beam_integration_test.go:65
    summary: HubOrchs' staging skip and its no-warning promise for an unwired pair have no test
    fix: Cover InWorktreeStaging with a small table test in internal/fabricengine over paths (nested under a .fabric-wt-staging-* directory, prefix-only name, unrelated path), and in TestHubOrchs_ListsWiredBeamsAndSkipsUnwiredPairs capture the logger output and assert it holds no warning for the unwired pair.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: consistency
    location: internal/loomengine/seed.go:123
    summary: VerifySeedOwnership still runs MkdirAll on the lock directory before its read, which the change otherwise removed from every reader
    fix: Delete the os.MkdirAll(filepath.Dir(statusLockPath)) call at the top of VerifySeedOwnership, and any import only it needed, since ReadJSON now reads without the lock when the lock directory is absent. Update the function's doc comment if it mentions the mkdir.
    origin: lens:code-correctness
---

### [MEDIUM] LoadState on a worktree with no scratch directory is untested

**Location:** internal/websterengine/state_test.go
**Issue:** The decision record's acceptance criteria require that `LoadState` on a worktree with no scratch directory returns no state and creates nothing.
The diff removes the `MkdirAll` that made the old path safe, but the existing `LoadState` tests create the scratch directory or save first, and none asserts that nothing is created.

### [MEDIUM] ReadOrigin's no-.weft, no-exclude behavior is unasserted

**Location:** internal/fabricengine/add_hubreader_integration_test.go:28
**Issue:** The hook runs `ReadOrigin` on the half-built pair but asserts only that no error returns and that `<warp>/_lym` is absent.
The acceptance criterion names no `.weft` left behind, and the behavior change in `ReadOrigin` (joining the lock path without creating `.weft` or seeding its exclude) is not pinned by any assertion.
The hook also fires after the weft worktree has moved to its anchor, so the staging case itself is not reached here.

### [LOW] The lock-error bound of ReadJSON is untested

**Location:** internal/state/strict_test.go
**Issue:** `TestReadJSON/LockDirectoryAbsent` pins the skip half of the decision record's bound.
No row pins the other half: a lock error that is not a not-exist error still fails the read.

### [LOW] HubOrchs' staging skip and silent skip are untested

**Location:** internal/orchcli/beam_integration_test.go:65
**Issue:** No worktree in the test sits under a `.fabric-wt-staging-*` directory, so the staging branch of `HubOrchs` is uncovered, and `InWorktreeStaging` has no test of its own.
The acceptance criterion that `HubOrchs` logs no warning for an unwired pair is not asserted, since the test does not capture the logger.

### [LOW] VerifySeedOwnership still makes a directory before a read

**Location:** internal/loomengine/seed.go:123
**Issue:** The board entry says a read never creates a directory, anywhere, and `internal/state/doc.go` now says the same.
`CheckSeed` dropped its `MkdirAll`, but `VerifySeedOwnership` keeps one before its `ReadJSON`.
It runs in a run's own worktree at bootstrap, not from the hub reader, so the race is not reachable, but the call is now redundant and contradicts the stated rule.

## Rejected

- Lens code-tests F6 (NIT, `TestStatusCmd_AbsentLockDirReadsWithoutCreatingIt` overlaps `TestReadJSON/LockDirectoryAbsent`): it covers the status verb's envelope at the CLI layer, a different surface, and the lens itself asked for no change.
- Lens code-tests F7 (LOW, the wire-before-marks-and-launchers order is not pinned): the decision record says neither the marks nor the launchers depend on the order, so the order is a window-shortening choice, not a behavior a test must hold.
- Lens code-correctness remark that `InWorktreeStaging` matches any ancestor directory with the prefix: unreachable in practice, and hub directories are not named that way.
- The `_lym/landing/summary.md` claims were checked against the diff and each is borne out; the one stated test name `TestReadJSON` with its four rows matches `internal/state/strict_test.go`.
