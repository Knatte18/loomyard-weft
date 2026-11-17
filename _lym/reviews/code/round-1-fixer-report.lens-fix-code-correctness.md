```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code in commits a7987cdb, c576bcd7 and 159fc358, through the fix-code-correctness lens. I found no BLOCKING or MEDIUM defects. Two LOW gaps in the tests.

**Traced and holding:**
- **`bootstrapCommitPaths` / `seedAndCommitBootstrap`:** A re-entry after a failed first-seed commit finds the pins file on disk, so `pinsStatErr == nil` and the pathspec carries `StencilPinsRel`.
  - An absent file leaves the pathspec as before, so git never sees a pathspec that matches nothing.
  - A stat error other than not-exist is treated as absent, which is harmless.
  - `bootstrapCommitPaths` has one caller, and `writeStencilPins` no longer returns a bool, so nothing was left broken.
- **`runDriverSpawnAndWait`:**
  - Every failure return is now `false, false`; the build is green.
  - Success returns `true, mustSpawn || parkedLive`. A live working driver, or a held driver lock with no strand, gives `false`, which is the F8 no-op.
  - A spawn and a parked-driver wake give `true`.
  - The remaining test call sites discard the second result with `ok, _ :=`. The bare call at `start_midmerge_test.go:360` is still valid.
- **`startSuccessFields`:** With the flag false it returns the plain envelope and posts nothing. With it true it behaves as before.
- **`succeedAfterWake` in `resume.go`:** It releases the lock, then runs `warnChangedStencils`, then `output.Ok`. Only the two driver-putting-to-work paths use it.
- **Added tests that fail with the defect in place:**
  - `TestStartSuccessFields`, row `WorkingDriverLeftAloneWarnsOfNothing`, fails if the warn becomes unconditional.
  - The resume parked-wake row fails if the check is removed from the wake path.
  - The failed-post row pins the one attempted notice and the still-listed stencil.
  - The unreadable-copy row (a directory where the board copy belongs) makes `ReadFile` return a non-not-exist error naming the stencil.
  - The new `putToWork` assertions in `start_driver_test.go` pin both the true and the false case.

**Findings:**

1. **LOW**, `sharedbootstrap.go`, `pinsStatErr == nil` passed to `bootstrapCommitPaths`. The caller's wiring of the pins commit is untested.
   - `TestBootstrapCommitPaths` pins only the pure helper.
   - The bootstrap integration test still runs under `FABRIC_SKIP_GIT`, so the commit is a no-op there.
   - Changing the argument to `false`, or dropping the stat, passes every test, including the F1 re-entry case.
   - The review's primary F4 fix, a git-backed pair asserting the pins file is tracked after the first seed, was not done. The fixer took the lighter option.
2. **LOW**, `stencilstore/pins_test.go`, row `EveryEditedBodyIsReportedInNameOrder`. It edits two stencils, so a `ChangedPins` that skipped `sort.Strings` would still pass about half the time, because map iteration order is random. A flaky pass is possible; a three-name row would detect the regression more reliably.

Further tool calls made: none. My context already held every changed file and the review.
