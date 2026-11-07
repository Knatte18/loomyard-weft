---
round: 1
focus:
  - "ensure_integration_test.go `staleHub` Build: does it hold the D7 and acceptance rules, with no `os.Setenv`, `os.MkdirTemp`, `git config --global`, kept `Context()`, package-global seam setter, or step that leaves the board's HEAD unequal to its upstream tip?"
  - "ensure_integration_test.go `staleHub` against the old `newStaleHub`: is the start state the same by construction, with the same step order and pair-a added by the registration's Base, and does `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` still break its config on the same state?"
  - "Tests that install a process-global seam (`logcapture.Capture`, `boardkit.InstallReplay`): are all copies taken before the install, in `TestEnsureOrWarn_WarnsWithTheWayForward`, `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent` and `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent`?"
  - "`TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent` and `arrangeUnpushedHubWide`: does taking both copies up front and passing them into the subtests keep each subtest on a fresh hub with its assertions unchanged?"
  - "D5: every remaining `hubforge.CopyHub(t, Shape{Anchor: \".\"})` site in the package, and the converted `NewHub` sites, against the rule that a site keeps `NewHub` only when its test is about the clone or wiring; is any kept `NewHub` site left without a named reason?"
  - "D8: is `TestEnsure_HeldLockTimesOutWithoutStamping` at tier 0 free of git, hub and process-global state, are all its assertions kept (error type, `Path`, no stamp written), and does `readStamp` check the same thing the removed `stampKey` did?"
  - "stamp_test.go: does the in-memory `Geometry{BoardDir, WorktreePath}` let `Ensure` reach the lock before any git or hub read, so the timeout is exercised and not a different early error?"
  - "The `t *testing.T` to `testing.TB` signature changes: are they needed by the Build function and nothing wider than that, with no helper left unconverted that the Build calls?"
  - "Commit messages and `_lym/landing/summary.md` claims against the diff, including 'none of their tests is about the clone or its wiring'."
---

The diff touches only `internal/hubreconcile`: two test files.
The package's `hubforge.NewHub` sites are all converted; the focus entries check that the conversion honors D5, D7 and D8 and the acceptance criteria on seed ordering.
