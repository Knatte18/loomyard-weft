# hubreconcile tests on a registered template, held-lock test at tier 0

The `internal/hubreconcile` tests now build their stale hub once per test binary and copy it per test.
They also stop cloning fresh hubs where a copy does the same job.

## Changes

- `staleHub` is a package-level `hubforge.Registration` with base shape `.` and the pair `pair-a`.
  Its build runs the steps `newStaleHub` ran after its copy, unchanged and in the same order:
  the pre-migration file set and the retired key, committed in the prime's and the pair's records worktrees.
  `newStaleHub(t)` returns `hubforge.CopyRegistered(t, staleHub)`.
  The slug parameter is gone, since every caller passed `pair-a`.
- The build commits only in the records worktrees and never on the board, so the board stays at its upstream tip as the registration requires.
  It sets no env, makes no temp dir and installs no seam.
  The helpers it calls (`commitPreMigrationFiles`, `seedRetiredKey`, `replaceOnce`) now take `testing.TB`.
- `TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp`, `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`, and `arrangeUnpushedHubWide` used `hubforge.NewHub(t, ".")`.
  They now take `hubforge.CopyHub` of the `.` shape.
  None of these tests is about the clone or its wiring, so D5's rule keeps no `NewHub` site in this package.
- A test that installs a process-global seam now takes its copies before installing it, so no template build runs under the seam:
  - `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent` copies both subtests' hubs before `boardkit.InstallReplay`, and `arrangeUnpushedHubWide` takes the hub as a parameter.
  - `TestEnsureOrWarn_WarnsWithTheWayForward` and `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent` start `logcapture.Capture` after their copy.
- `TestEnsure_HeldLockTimesOutWithoutStamping` moves from `ensure_integration_test.go` to `stamp_test.go` and runs at tier 0.
  Its geometry is two temp dirs with no hub and no git.
  An absent stamp sends `Ensure` straight to the lock, which another holder keeps.
  It keeps both assertions: the error is a `*LockTimeoutError` naming the lock path, and no stamp is written.

## Design decisions

- `pair-a` stays a literal in the registration, matching the literal every test in the file already uses for the pair.
- The held-lock test sits in the internal-package `stamp_test.go`, the tier 0 file that already covers the lock path, and reads the stamp through `readStamp`.

## Testing

`go vet ./internal/hubreconcile` passes.
The tests keep their assertions; the verify gate runs tiers 0 and 1.
