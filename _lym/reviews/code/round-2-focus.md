---
round: 2
focus:
  - "In internal/gatecli/test_integration_test.go, check that the orphan scenario drives a shell that exits 0 while an orphan holds the pipe, and settle whether removing the nil mapping of exec.ErrWaitDelay in runTiedChild (internal/gatecli/child.go) would fail a test."
  - "In internal/gatecli/test_integration_test.go runStandIn, check that the shared build invocation can start an orphan, and settle whether passing a zero pipe wait delay in runBuild (internal/gatecli/build.go) would fail a test."
  - "In _lym/landing/summary.md, check that the stated reason a reused group id is safe for proc.KillTree matches what killTreePlatform in internal/proc/tied_linux.go does."
---

Each entry names where a round-1 gap was reported and the question that settles whether the current tree closes it.
