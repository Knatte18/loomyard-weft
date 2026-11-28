---
round: 1
ledger:
  - key: loom-start-refusal-way-forward-cannot-clear
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: decision-verbs-name-record-write-row-on-jacquard
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: jacquard-start-after-own-motif-run-refused
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: profiles-reconcile-broken-default-and-uncovered-profiles
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tapestry-jacquard-equal-top-level-keys-contradicts-entry
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: gated-last-block-empty-on-done-skip-enters-block
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: loom-start-record-check-refuses-re-entry
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: plan-and-build-recipes-note-conflicts-with-one-jacquard
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: jacq-nickname-write-handed-to-parent
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d11-review-criterion-unreachable-at-code-review
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-rationale-overstates-preflight-omission
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d9-usage-bound-misses-discussed-entry-without-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 is the first round, so every key is new and open.

The gating findings fall in these areas:

- The refusals a jacquard run meets: their ways forward, the row names they cite, and where the start check sits.
- The path from an entry's own motif run to a jacquard start.
- The `profiles.yaml` reconcile on hubs whose profiles differ from the shipped ones.
- The new empty `on_done` exit when the last block is gated.

The two `decision` keys wait on the run's parent.
