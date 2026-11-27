---
round: 2
focus:
  - "internal/hubreconcile/testmain_integration_test.go, the seams TestMain installs for the whole test binary, the board-sync recorder (SetSpawnBoardSyncForTest) among them: does each stand-in or wrapper meet PATTERN-test-stand-ins, and does a recording wrapper pass every call to the module that fills the seam in production?"
  - "internal/hubreconcile/ensure_integration_test.go against its version at 2326ef4da0640800959a1d7b2bc9c3444f708748: for each assertion the range removes or widens, EnsureOrWarn's behaviour after a successful reconcile included, does another test still hold it on the same behaviour and no looser, or does a Loosened-Check trailer name it with a measured cause?"
  - "_lym/landing/summary.md and the commit messages of 2326ef4da0640800959a1d7b2bc9c3444f708748..HEAD: do the net test-line figures, the claims about removed retry walks and the claims about which tests or subtests run serially match the diff of the whole range as it stands at HEAD?"
---

The entries point at the test binary's shared setup, the coverage the cut removes,
and the accuracy of what the branch says about itself.
