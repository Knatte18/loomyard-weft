```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness over the `internal/hubreconcile` test diff.

**Per function or branch, with the case I ran**

- **`staleHub.Build`:** correct.
  - The build runs `commitPreMigrationFiles` and `seedRetiredKey` on the prime's and `pair-a`'s records worktrees. It makes no board commit, so the board's HEAD stays equal to its tip in the records bare and `requireBoardAtUpstreamTip` passes (`template.go:228,242-251`).
  - The step order matches the old `newStaleHub`: `pair-a` is added, then the pre-migration files, then the retired key, each in the prime and then the pair. `buildTemplateIn` adds `Base.Pairs` before `Build` (`template.go:222-227`).
  - The build calls no env setter, temp dir, `Cleanup`, global git config or seam setter. Helpers that fail inside it go through `templateBuildTB.Fatalf`, which records the failure and ends the build, so a failing step cannot leave a half-built template cached as good. `TestMain`'s `os.Setenv` runs before the tests, not in the build.
- **`newStaleHub(t)` and its callers, including `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas`:** correct.
  - Each caller gets its own relocated copy, so a row's `breakConfig` edits are not shared across parallel subtests.
  - The start state is the same by construction, so the broken-config rows act on the state they did before.
  - A copy folds the binding into the board's HEAD by amend and force-push. That changes the board's commit ids, not its tree. No assertion in the file reads a board commit id, and the counts compared are deltas.
- **`TestEnsureOrWarn_WarnsWithTheWayForward`:** correct.
  - `newStaleHub(t)` runs before `logcapture.Capture(t)` in every subtest, so a lazy first build never runs under the capture.
  - The test is non-parallel, so no parallel test builds a template while the capture is installed.
- **`TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent`:** correct. `CopyHub` runs before `Capture`. `set-url` on the copy's own origin then makes the remote unreachable, as it did on a fresh hub.
- **`TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp` and `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile` (NewHub to CopyHub):** correct. Both remove `origin` on the copy and assert commit deltas and file contents. The copy's board is clean against its HEAD and upstream, as a fresh hub's is.
- **`TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent` with `arrangeUnpushedHubWide(t, h)`:** correct.
  - Both copies are taken on the parent `t` before `boardkit.InstallReplay`.
  - Each subtest gets a distinct, fresh copy, so subtest 2 does not inherit subtest 1's pushed or replayed state.
  - `arrangeUnpushedHubWide` is unchanged apart from taking `h`: seed, push, two walks, then remove the stamp.
  - The `-u` push onto a branch already force-pushed by the copy is a no-op.
  - The parent's `TempDir` and teardown lifetimes cover both hubs.
  - Running only one subtest with `-run` still makes both copies. That costs time and breaks nothing.
- **`TestEnsure_HeldLockTimesOutWithoutStamping` (`stamp_test.go:85-108`):** correct.
  - The geometry is `Geometry{BoardDir: t.TempDir(), WorktreePath: t.TempDir()}`. `ensure` skips the pair branch, `readCurrentStamp` finds no stamp, and `current` is false, so `repoDefaultAbsent` is short-circuited (`walk.go:43`).
  - It then does `MkdirAll` on the lock dir and `AcquireWriteLockWithin`. The held flock times out after 50 ms and returns `*LockTimeoutError{Path: geom.LockPath()}` (`walk.go:55-61`).
  - `walkHub` and its git reads are never reached, so the error is the lock timeout and not an earlier one.
  - `readStamp(geom.StampPath())` on an absent file gives `found == false, err == nil`, which is the same condition the old `stampKey` checked. A parse error would now also fail the assertion, which is stricter, not weaker.
  - `t.Cleanup` releases the lock after the assertions, replacing the old `defer`. The `err` inside the `if` shadows the outer one and is correct.
- **`testing.TB` signature changes (`seedRetiredKey`, `replaceOnce`, `commitPreMigrationFiles`):** correct. These are the helpers the Build calls, directly or through `commitPreMigrationFiles` → `replaceOnce`. Each calls only `Helper`, `Fatalf` and `gitkit` functions that accept `TB`. `formerUnits` and `preMigrationBase` are not reached by the Build and keep `*testing.T`.

No findings.

**Tool calls:** none. My context already held the diff, `walk.go` `ensure`, `hubreconcile.go` and `template.go`.
