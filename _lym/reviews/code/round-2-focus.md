---
round: 2
focus:
  - "internal/loomshed/verifygate.go, failuresSection and outputAboveHeader: when two packages fail a test of the same name, does each failure quote its own package's output, and does the upward scan stop at a previous package's closing line (`ok  \t`, `FAIL\t`, `?   \t`)?"
  - "internal/loomshed/verifygate.go, failuresSection against verifyFailuresBytes: is the first failure cut to the byte bound, and do the tests cover the bound and the More failures line?"
  - "internal/gatecli/run.go runGreenAware and runTest, internal/gatecli/tierrun.go retryOnShortage: does a test drive runTest through a retry that passes and through a twice-failed shortage, so removing the retry call, the shortage propagation or the failedRunLabel call fails a test?"
  - "internal/gatecli/test_integration_test.go: are the build step's build-server variables, a tier env override of them and a subdirectory tier dir under a subdirectory anchor asserted, and is the context-cancelled-during-retry-delay branch of retryOnShortage covered?"
---

Focus restates the open round 1 areas so a fresh reviewer re-checks the fixed code.
