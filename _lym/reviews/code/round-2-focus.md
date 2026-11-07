---
round: 2
focus:
  - "Check that `computeScope` in internal/gatecli/scope.go, for the `--card` scope, fills the watch skip (`cardChanges`, `skipNames`, `sc.skip`) and that a test in internal/gatecli fails when that skip breaks, including a worktree with no parent branch."
  - "Check that `scanGuards` in internal/gatecli/scope.go drops a guard test that also carries `//lym:watch` when no changed path concerns it, and that a test covers `moduleRelative` for a module dir that is not the worktree root."
  - "Check that `armDeadline` in internal/gitexec/gitexec.go complies with the 'Time is injectable' rule of pattern/PATTERN-test-speed.md, or that the pattern text states the exception, and that a test pins the production deadline killing a hung remote."
  - "Check that `TestRun_Cancellation` in internal/verifyrun/run_integration_test.go cancels only after the long-running child has started."
  - "Check that the guard section of pattern/PATTERN-test-speed.md matches `scanGuards` and `targetsFor` for a test carrying both `//lym:guard` and `//lym:watch`, and that `guardScanRefusal` in internal/gatecli/cli.go states the way forward once."
---

Targets the sites round 1 found weak; the reviewer still reads the whole diff.
