---
round: 1
ledger:
    - key: origin-wait-first-pass-unbounded-respawn
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: beam-catchup-wait-missing-from-push-summary
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: non-origin-catchup-failures-lose-start-refusal
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: design-wrong
    - key: doc-notice-contents-omit-catchup-line
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: waitingsince-at-fallback-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: since-reset-on-reason-change-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: summary-origin-only-filter-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: catchup-line-under-runs-heading
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
Round 1 is the first round, and every key is open.

The two gating keys concern what the change's transient-stop handling leaves unbounded or unreported:
a first pass that exits as an origin wait can be started again with no backoff or notice,
and the push summary reads only the catch-up wait of the dobby it runs in.
The scope key concerns the same exit code covering catch-up failures other than an unreachable origin,
which also widens the respawn case.
The remaining keys are a doc omission, missing tests and a notice heading.
