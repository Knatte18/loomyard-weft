---
round: 1
ledger:
  - key: duplicated-logcapture-helper
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: reconcile-hubconfig-warpbinding-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: pushrecorded-retry-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: hubwide-unpushed-commit-stamp-one-run
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: stale-modefor-comment-smoke-switchseed
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised five findings: F1 maps to `duplicated-logcapture-helper`, F2 to `reconcile-hubconfig-warpbinding-failure-untested`, F3 to `pushrecorded-retry-failure-untested`, F4 to `hubwide-unpushed-commit-stamp-one-run` and F5 to `stale-modefor-comment-smoke-switchseed`.
All are open.
The BLOCKING `PATTERN-testkit` finding is what keeps the loop going.
