---
round: 1
ledger:
  - key: partial-scope-no-reinterpretation
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: recorded-partial-scope-inflight
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: master-no-fixer-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: post-writer-regression-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: retired-keys-not-refused
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: command-mode-judge-picks-cases
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: seal-check-command-mode-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: reentry-seals-interrupted-expect
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: restore-wrong-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: judge-snapshot-misses-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: past-drift-expect-test-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: amendment-reach-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: gate-seam-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: drift-verify-gate-key-orphaned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: past-recheck-lands-empty-verdict
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: passing-check-command-mode
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: d6-acceptance-hand-edited-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-range-statements-unread
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: baseline-lost-before-expect
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: identity-hash-sign-ambiguity
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: open-range-bound-missing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rationale-reargues-rejected
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

# Ledger, round 1

First ledger: every finding of round 1 is open, keyed by its identity.

- Config and in-flight runs: `partial-scope-no-reinterpretation` (BLOCKING), `recorded-partial-scope-inflight`, `retired-keys-not-refused`, `past-recheck-lands-empty-verdict`.
- Seal and expectation: `reentry-seals-interrupted-expect`, `restore-wrong-way-forward`, `past-drift-expect-test-unstated`, `seal-check-command-mode-contradiction`, `d6-acceptance-hand-edited-record`, `identity-hash-sign-ambiguity`, `baseline-lost-before-expect`.
- Judge and command mode: `command-mode-judge-picks-cases`, `judge-snapshot-misses-commit`, `amendment-reach-bound`, `passing-check-command-mode`, `open-range-statements-unread`, `open-range-bound-missing`.
- Gate placement and way forward: `master-no-fixer-premise`, `post-writer-regression-no-way-forward`, `gate-seam-overstated`, `drift-verify-gate-key-orphaned`.
- Relocation: `rationale-reargues-rejected`.
