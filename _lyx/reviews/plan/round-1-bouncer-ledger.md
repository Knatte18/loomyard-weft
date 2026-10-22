---
round: 1
ledger:
  - key: push-retry-owner-guard-unlicensed
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: adoptcli-cli-cobra-deviation-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: setowners-retry-delay-not-injectable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: innerrun-owner-trigger-lost-at-call-boundary
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: resume-prime-claim-failure-undecided
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: pattern-short-line-drops-exception
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: machinename-leaf-docs-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: unrecorded-pair-bypass-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cross-package-symbols-bundled
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: adopt-step2-reword-departs-d5
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: childspawnerror-unreachable-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings and no earlier ledger exists.
Gating: the unlicensed push-retry owner guard (F1), the unlisted `adoptcli` deviation (F2) and the non-injectable `SetOwners` retry delay (F3) are BLOCKING; the InnerRun owner trigger lost at a Call boundary (F4) is a MEDIUM design finding.
The rest are MEDIUM or LOW and non-gating.
