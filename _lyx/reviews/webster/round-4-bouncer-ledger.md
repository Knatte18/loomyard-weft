---
round: 4
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
    status: resolved
---

Rounds 1 and 2 raised the serial publish-retry tests, the reason-prefix pause rule and the hand-rolled logger capture in the teardown test; none returned.
Round 3 raised one BLOCKING `design` finding, a compile error from the old two-argument `noticeDriverBusy` call in `publishRetryResume`.
Round 4 found the call now passes the spawn error, with build, vet and the `battenshed` gate clean, and raised no findings.
