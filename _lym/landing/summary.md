# hubreconcile tests: registered stale hub, CopyHub, tier-0 lock test

Speeds up the `internal/hubreconcile` tests.
The change is test-only; no production code is touched.

## What changes

- `newStaleHub` is now built once per test binary from a `hubforge.Registration` (`staleHub`) and each test takes its own copy with `hubforge.CopyRegistered`.
  The template holds the `pair-a` pair, the pre-migration file sets with the retired key committed, and no stamp.
  The build steps are the old fixture's, moved as they were, so the start state is unchanged.
  `newStaleHub` loses its `slug` parameter, since every caller passed `pair-a`.
- The remaining `hubforge.NewHub` sites in `ensure_integration_test.go` take `hubforge.CopyHub` of the same anchor.
  None of those tests is about the clone or the wiring, so no site keeps `NewHub`.
- `TestEnsure_HeldLockTimesOutWithoutStamping` moves to `stamp_test.go` at tier 0.
  It runs on a `Geometry` over temp directories instead of a hub, with its assertions kept.
  The no-stamp check reads the stamp through `readStamp`.
- `seedRetiredKey`, `replaceOnce` and `commitPreMigrationFiles` take `testing.TB`, since the registration's build receives a `testing.TB`.

## Design decisions

- The registration's build commits only in the records worktrees, so the board stays at its upstream tip.
- Tests that install a process-global seam take their copies first.
  `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent` takes both hubs before `boardkit.InstallReplay`, and `arrangeUnpushedHubWide` now receives the hub.
  The two tests that capture the process-global log likewise take their hub before `logcapture.Capture`.

## To check

- Nothing was run by hand beyond the package's own tests.
