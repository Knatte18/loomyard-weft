---
round: 1
ledger:
  - key: verifier-closes-unfixed-blocking-key
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: verify-test-no-partial-close-row
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: verify-test-spawn-error-and-write-failure-rows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: verify-test-ledger-rewrite-preservation
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: verify-test-fixed-blocking-selection-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: verify-test-parse-refusals-and-ondisk-rerun
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hard-cap-halt-leaves-verification-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: ledger-rewrite-not-atomic
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rejudge-doc-claims-judge-sole-writer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verification-format-missing-from-sole-parsers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: below-window-accept-blocking-unrecorded-reading
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: parse-verification-accepts-duplicate-key
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, none BLOCKING.
The gating ones are F1 (Go does not restrict which BLOCKING keys the verifier may close) and the test gaps F2 and F3.
F4 to F6 are further coverage gaps, F8 an atomicity gap and F12 a duplicate-key refusal.
F7, F9 and F10 are consistency fixes, and F11 is deferred to the parent as an unrecorded reading.
