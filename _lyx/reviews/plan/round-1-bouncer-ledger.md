---
round: 1
ledger:
  - key: plan-write-rotation-refusal-rows-missing
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: begin-batch-duplicate-table-test
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: reset-targets-test-misplaced-and-doubled
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: fingerprint-way-forward-callers-misstated
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: tokencount-startfit-write-breaks-build
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: standalone-reset-dirty-path-refusal-unspecified
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: refusal-spec-existing-rows-left-stale
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: reconcile-migration-duplicate-assertions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: merriam-base-computed-on-recorded-partition
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: reset-keep-helper-in-wrong-card
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: reconcile-docs-not-in-card-targets
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: webster-yaml-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: preflight-anchorpath-entry-table-row-missing
    rounds: [1]
    status: open
    class: scope
    severity: NIT
  - key: archive-only-path-runs-no-git-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first review of the plan, with 14 findings and no prior ledger.
The BLOCKING findings are a missing refusal-spec row set for Plan-Write rotation (F1) and two test-economy breaches in the begin-batch and reset cards (F2, F3).
The one gating design finding is the loose description of refusal-spec row edits (F7).
The key-to-finding mapping follows the review's order: F1 to F14.
