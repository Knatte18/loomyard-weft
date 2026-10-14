---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-run-settings-refusal-mechanism
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d9-handover-ends-hold
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d9-blocked-eligibility-source
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refusal-spec-row-by-number
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's nine keys are all resolved: round 2 did not re-raise any, and its focus verdicts clear D9's cause, D2 and D1, D5 and D4's both-keys outcome.
Round 2 raised five new keys, all around the D9 hold and D4's `run_settings` path.
The gating one is the BLOCKING `d9-post-hold-relay-wait`: after the hold ends on a hand-on, `stateChangeReady` sends the notice at once, so the double notice returns.
`d4-run-settings-refusal-mechanism` needs a decided place where the refusal is raised, and `d9-handover-ends-hold` needs the end condition to ignore a hand-over's report and marker.
The last two are local corrections to D5/D9 wording and to a row cited by number.
