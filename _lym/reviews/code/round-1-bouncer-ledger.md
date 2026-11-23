---
round: 1
ledger:
  - key: orch-contract-names-unsent-no-parent-mail
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: summary-claims-deleted-answerparentask-wrapper
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: recovered-converged-test-lacks-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-answerparentask-references-in-tests
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: empty-tag-guard-has-no-pinning-row
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings, none gating: one MEDIUM consistency, two LOW consistency, one NIT consistency and one NIT design.
Each fix is a local correction whose text the review already fixes: a contract line, a summary sentence, a keep marker, renamed messages and one test row.
