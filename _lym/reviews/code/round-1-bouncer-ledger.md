---
round: 1
ledger:
  - key: unrecorded-departures-git-only-guard-and-open-hunt
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: main-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fallback-message-misreports-two-cases
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-test-row-for-first-watched-dir
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-symlink-test-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: message-order-depends-on-map-iteration
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, all in `internal/testkit/tmuxkit`.
F1 (decision, MEDIUM) is deferred to the parent.
F2 (Main's wiring untested) is the one gating finding.
F3 to F6 are small message, test-row and ordering corrections.
