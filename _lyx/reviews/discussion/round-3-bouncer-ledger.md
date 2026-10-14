---
round: 3
ledger:
  - key: d9-root-cause-deferred
    rounds: [1]
    status: resolved
  - key: ac-hold-ending-mismatch-d9
    rounds: [1]
    status: resolved
  - key: d2-loom-upsert-check-premise
    rounds: [1]
    status: resolved
  - key: d5-unreadable-record-contract
    rounds: [1]
    status: resolved
  - key: d4-both-keys-migration
    rounds: [1]
    status: resolved
  - key: d4-migrate-contract-widening
    rounds: [1]
    status: resolved
  - key: scope-missing-shedadapters
    rounds: [1]
    status: resolved
  - key: orch-stencil-change-only-in-notes
    rounds: [1]
    status: resolved
  - key: d6-commit-history
    rounds: [1]
    status: resolved
  - key: d9-post-hold-relay-wait
    rounds: [2]
    status: resolved
  - key: d4-run-settings-refusal-mechanism
    rounds: [2]
    status: resolved
  - key: d9-handover-ends-hold
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-blocked-eligibility-source
    rounds: [2]
    status: resolved
  - key: refusal-spec-row-by-number
    rounds: [2]
    status: resolved
  - key: d9-stop-report-premises
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d9-hold-end-time-source
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Rounds 1 and 2 keys are resolved except `d9-handover-ends-hold`.
Round 3 clears the post-hold relay wait, D4's `run_settings` refusal and the single eligibility function.
`d9-handover-ends-hold` returns at MEDIUM: the round-2 identifying clause also matches the successor driver's own park marker, so a tackler hand-back at the same stop leaves the hold on until the backstop.
Two new LOW keys sit on D9: the end condition and criterion name a stop report batten never reads, and the hold's end time that the report wait starts from has no named source across a batten restart.
