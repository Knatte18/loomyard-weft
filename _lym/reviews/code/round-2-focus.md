---
round: 2
focus:
  - "internal/gatecli/strayfiles_integration_test.go: does the file carry a `//go:build !windows` constraint and no run-time `runtime.GOOS` skip, per PATTERN-test-speed's rule that an OS-bound test is left out of the other OS's build?"
  - "internal/gatecli/copies.go addFailedRuns and checkStrayFiles in internal/gatecli/strayfiles.go: on a `--copies` run that leaves a stray file, does the failure report list the stray-file cause exactly once, both when every copy passed and when a copy failed otherwise, and does a test reach that case?"
  - "internal/gatecli/tierrun.go runTier and internal/gatecli/run.go runGreenAware around retryTier: is the untracked-file listing taken once per tier run, so a file left by a failed first attempt still counts as new when a rerun passes, and does a test pin it?"
  - "internal/gatecli/tierrun.go refusal skip in runTier: does a test reach a refusal after a successful listing and assert no stray-file line is added?"
  - "internal/gatecli/strayfiles.go strayFilesLine: are names holding a newline or a comma written so the stray-file line stays one line with unambiguous paths, and do the tests assert it?"
  - "internal/gatecli/strayfiles_integration_test.go truncation row: does it pin the cap from literal names rather than from the `strayFilesListed` slice the code uses?"
---

Focus names where to look after the round-1 fixes; it caps no severity and pre-states no verdict.
