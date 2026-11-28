---
round: 1
ledger:
  - key: primeshuttle-absent-pair-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: other-remote-lookups-not-retried
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: summary-understates-prime-lock-hold
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-test-for-non-transport-no-retry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: backoff-hook-fatalf-off-test-goroutine
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unread-wantsleeps-table-value
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised its findings in two areas.
In `internal/battencli`, the quiet absent-pair path of `primeShuttle` lacked a test; this is the gating finding.
In the pair create's retry of remote lookups, the findings cover the lookups left without a retry,
the change description's figure for the prime-lock hold, and the retry test's coverage and construction.
