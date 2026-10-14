---
round: 4
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
    status: resolved
  - key: d9-blocked-eligibility-source
    rounds: [2]
    status: resolved
  - key: refusal-spec-row-by-number
    rounds: [2]
    status: resolved
  - key: d9-stop-report-premises
    rounds: [3]
    status: resolved
  - key: d9-hold-end-time-source
    rounds: [3]
    status: resolved
  - key: d9-hand-on-end-time-file
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-hand-on-episode-bound
    rounds: [4]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d9-report-wait-reason-circular
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: d9-applied-resume-record-premise
    rounds: [4]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Rounds 1 to 3 keys are all resolved: round 4 no longer raises the hand-over exemption or the stop-report wording, and its focus verdicts confirm both.
Round 4 is narrowed to D9's hand-on end condition.
F1 (`d9-hand-on-end-time-file`, MEDIUM) says the hand-on end is dated by `verdict.yaml`, which timeouts, deaths, missing or unparseable verdicts and failed resume preconditions never leave; `envelope.json` is the record every outcome writes.
F2 (`d9-hand-on-episode-bound`, LOW) says a same-key re-halt ends the new hold at once on an old attempt's hand-on.
F3 and F4 are NITs on the report wait's stated reason and on the claim that an applied resume leaves no dated record.
