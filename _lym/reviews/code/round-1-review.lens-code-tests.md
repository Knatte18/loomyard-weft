```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests — per behaviour, the test that pins it or none, plus Test-economy findings.

**Per behaviour**

- **`ReadJSON` creates no directory, and a missing data directory reads as not found.** Pinned by `TestReadJSON/MissingDirectory` in `internal/state/strict_test.go`. It stats the absent dirs after the read.
- **`ReadJSON` reads without the lock when the lock directory is absent.** Pinned by `TestReadJSON/LockDirectoryAbsent`. It reads a present record and checks that the lock directory stays absent.
- **`ReadJSON` skips the lock only on a not-exist error and fails on any other lock error.** None. No row makes lock acquisition fail with something other than not-exist, such as the lock path being a directory or an unreadable lock directory. The decision record states this bound, but only the skip half is tested. (Finding F1.)
- **A read failure other than not-exist wraps `ErrRead`.** Pinned by `TestReadJSON/UnreadableFile`, where the data path is a directory.
- **A decode failure wraps `ErrDecode`.** Pinned by `TestReadJSON/MalformedJSON` and by `TestCorruptFile` in `internal/state/state_test.go`. These overlap: `TestCorruptFile` also asserts `errors.Is(err, state.ErrDecode)`, but it additionally pins the `UpdateJSON` abort, so it is not redundant.
- **`readJSONUnlocked` now wraps `ErrRead` for `UpdateJSON`'s read.** None. `UpdateJSON` shares the function, but no `UpdateJSON` test asserts `ErrRead`. I did not open the `UpdateJSON` tests (tool-call budget), so call this unverified. It is low value.
- **Removal of `TestMissingFile`.** Its old assertion that `ReadJSON` creates the parent directory and lock file is now false, and `TestReadJSON/MissingDirectory` replaces it. The removal is correct, not a coverage gap.
- **`ReadJSONNoMkdir` removed, with no reference left.** No test. The compiler and the verify gate cover it, and the repo grep earlier showed no match.
- **`ReadOrigin` creates no `.weft` directory and seeds no exclude.** Pinned only indirectly by `TestAdd_HubReaderBeforeWiringLeavesNoLym` in `internal/fabricengine/add_hubreader_integration_test.go`. In that test `ReadOrigin` runs on the half-built pair at the hook, but the assertions are that `_lym` is absent and that no error comes back. Nothing asserts that no `.weft` directory or exclude entry appears under the weft worktree or the staging directory. The decision record's acceptance criterion says "no `.weft` under a staging directory", and the test does not assert it. (Finding F2.)
- **`WriteOrigin` keeps the creating path.** Not newly pinned, but the diff does not change it, and the existing `Add` tests cover it.
- **`InWorktreeStaging`.** Its positive case has no test. In the new fabricengine test the hook fires after the weft worktree has been moved to its anchor, so `List` probably shows no staging path. In `TestHubOrchs_ListsWiredBeamsAndSkipsUnwiredPairs` in `internal/orchcli/beam_integration_test.go`, no worktree sits under a `.fabric-wt-staging-*` directory. The "skip a worktree under staging before resolving it" branch in `HubOrchs` is therefore uncovered. (Finding F3.)
- **`LymJunctionWiredHere`, and `HubOrchs` skipping an unwired pair silently.** Pinned by `TestHubOrchs_ListsWiredBeamsAndSkipsUnwiredPairs`. The test removes the `_lym` link and asserts the pair is unlisted and no `_lym` is recreated.
- **`HubOrchs` logs no warning for an unwired pair.** None. The test does not capture the logger. The acceptance criterion says "logs no warning for it". (Finding F4.)
- **`HubOrchs` still lists a wired beam, with the prime first.** Pinned by the same test (`want {prime, "wired"}`).
- **`LymJunctionWiredHere` error path** (an `IsLink` error leading to a warning and a skip). None. This is an unlikely path, so I note it only.
- **`Topology.Add` wires junctions right after the weft worktree and before the local marks and launchers.** Pinned for "junctions are wired by the end" only. `TestAdd_HubReaderBeforeWiringLeavesNoLym` shows the link exists after `Add` and the `_lym` is absent at the hook. Nothing asserts the order against `MarkBranchLocal` or `writeLaunchers`. The test uses `SkipPush: true`; whether `local` is true there I did not verify. The ordering is behaviour the decision record claims, though it is only a shortened window, so I rate it low.
- **`Add` rollback still works when wiring fails.** The failure path is unchanged in logic (same `rollbackAdd` calls), moved earlier. Existing rollback tests are presumed to cover wiring failure, but I did not open them. Unverified.
- **`Add` completes after the hook.** Pinned by the `IsLink` assertion at the end of the new test.
- **`websterengine.LoadState` on a worktree with no scratch directory returns `(nil, nil)` and creates nothing.** None. The existing `LoadState` tests (`state_test.go`, `archive_test.go`, `runlevel_test.go`) create the scratch directory or save first. `TestLoadState_UnusualFiles` pins an absent `state.json`, but I did not confirm its scratch directory is absent, and it makes no "creates nothing" assertion. The acceptance criterion is stated explicitly. (Finding F5.)
- **`checkRunStatus`, `CheckSeed` and the `status` verb no longer mkdir the lock directory.** `TestStatusCmd_AbsentLockDirReadsWithoutCreatingIt` pins the `status` verb: the read reaches found=false and the parent stays absent. `checkRunStatus` and `CheckSeed` have no new test. Their mkdir removal is covered indirectly by `TestReadJSON/LockDirectoryAbsent`, so I do not count it as a gap.
- **The `EnsureStatusLockDir` field's effect on `status`.** The old test `TestStatusCmd_EnsureStatusLockDir_False` is deleted. The new test sets the field to true, according to its comment, and shows that `status` ignores it. This is fine.

**Test-economy**

- No new test duplicates an existing one by name.
- `TestStatusCmd_AbsentLockDirReadsWithoutCreatingIt` partly duplicates `TestReadJSON/LockDirectoryAbsent`. The former adds the `status` verb's found=false envelope and the lock-less read at the CLI layer, which is a different surface, so I keep it. (Finding F6, NIT, optional.)
- `TestReadJSON/UnreadableFile` puts a directory at the data path. That is fine on Linux. On Windows `os.ReadFile` on a directory also errors, so the row is probably portable.

**Findings**

- **F1 — MEDIUM, design:** `internal/state/strict_test.go`, around the `TestReadJSON` rows. No row pins the "any other lock error still fails the read" bound. A regression to "skip the lock on any lock error" would pass every test. Add a row where the lock path is a directory, or the lock directory is unreadable, expecting an error that is not the not-exist skip. The probable fix is a row with a lock path that cannot be opened but whose parent exists. The lock file is a regular file path, so a dir at the lock path should make `flock` fail with something other than ENOENT.
- **F2 — MEDIUM, design:** `internal/fabricengine/add_hubreader_integration_test.go`, in the hook. The acceptance criterion "no `.weft` under a staging directory" and the claim that `ReadOrigin` seeds no exclude are asserted nowhere. Add an assertion in the hook that the weft worktree has no `.weft` directory after the read, and that its exclude is unchanged. A `ReadOrigin` unit test over a pair with no `.weft` would also do.
- **F3 — LOW, scope:** `internal/orchcli/beam_integration_test.go`, `TestHubOrchs_ListsWiredBeamsAndSkipsUnwiredPairs`. The staging skip has no test. Add a row for `InWorktreeStaging` (a table over paths: nested, prefix-only, unrelated) as a tier0 unit in `internal/fabricengine`, which is cheap.
- **F4 — LOW, scope:** `internal/orchcli/beam_integration_test.go`. "Logs no warning for an unwired pair" is unasserted. Capture the logger output in that test.
- **F5 — MEDIUM, scope:** `internal/websterengine/state_test.go`. `LoadState` with an absent scratch directory has no test: it returns `(nil, nil)` and creates nothing. Add a case to `TestLoadState_UnusualFiles` or a small test that points `scratchDir` at a nonexistent path and stats it afterwards.
- **F6 — NIT, design:** `internal/shedverbs/status_test.go`. `TestStatusCmd_AbsentLockDirReadsWithoutCreatingIt` partly overlaps `TestReadJSON/LockDirectoryAbsent`. It is acceptable because it covers the CLI envelope; no change is required.
- **F7 — LOW, design:** `internal/fabricengine/add_hubreader_integration_test.go`. The ordering "wire before marks and launchers" is not pinned. Optionally, in the hook, assert that no launcher file exists yet.

**Tool calls**

1. `git diff … -- internal/state/strict_test.go internal/shedverbs/status_test.go`: the manifest lacked the full test diffs for those two files.
2. `grep LoadState` in `internal/websterengine` tests, and `grep InWorktreeStaging|LymJunctionWiredHere` across `internal`: to find the existing tests and callers of the new predicates.
3. `sed -n 95,135p internal/websterengine/state_test.go` plus the `git diff --stat` of the webster files: to check whether `LoadState` tests cover an absent scratch directory, and that the diff changed no webster test.

I used 3 tool calls of the 6 allowed.
