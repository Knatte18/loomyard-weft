---
round: 3
ledger:
  - key: publish-retry-tests-serial
    rounds: [1]
    status: resolved
  - key: pause-detected-by-reason-prefix
    rounds: [1]
    status: resolved
  - key: teardown-test-handrolled-log-capture
    rounds: [2]
    status: resolved
  - key: notice-driver-busy-missing-cause
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
---

Rounds 1 and 2 raised the serial publish-retry tests, the reason-prefix pause rule and the hand-rolled logger capture in the teardown test; round 3 does not raise any of them again.
Round 3 raised one BLOCKING `design` finding: after the merge-in, `publishRetryResume` calls `noticeDriverBusy` with its old two-argument form, so `battenshed` fails to compile and the Publish verify fails.
