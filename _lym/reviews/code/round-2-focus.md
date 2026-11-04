---
round: 2
focus:
  - "internal/websterengine/verifygate.go and internal/webstercli/verify.go: with a verify result whose RunnerCrash is set, does the webster verify gate return a terminal failing result that names the crash and spawns no verify fixer fork or cold fixer, and does `lym webster verify` report the crash with the way forward instead of a failure to fix? PATTERN-gate-fixer says a runner crash gets no fixer."
  - "Tests for the crash paths: for each of the runTest closing-line rule, exitCodeOf's signal branch and detachRun.endedOnRunnerCrash in internal/gatecli, Verify's reading of the runner crash line in internal/verifytree/verify.go, the Parent-Align crash halt and the merge fixer's crash end in internal/landingshed, and Terminal from RunnerCrash in internal/loomshed/verifygate.go, does a test fail if that behavior breaks, and does any new test duplicate an existing one?"
  - "internal/gatecli/tierrun.go retryTier: is a rerun crash labelled a runner crash when the first run was a shortage rather than a crash?"
  - "internal/landingshed/verifygate.go and prreview.go: what does the pr-review gate do on a repeat runner crash, and does it stop with the crash named instead of asking the agent to end its turn again?"
  - "internal/gatecli/run.go exitCodeOf is shared with the build step: is a build ended by a crash signal described in the Exit codes paragraph of internal/gatecli/doc.go?"
---

Focus lists the open round 1 questions as fresh questions for the next reviewer.
