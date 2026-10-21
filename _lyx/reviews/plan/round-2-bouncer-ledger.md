---
round: 2
ledger:
  - key: new-keys-break-exact-set-pins-and-docs
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card2-generation-start-skip-no-bound
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: subjectless-bouncer-writes-generation-record
    rounds: [1]
    status: resolved
  - key: rework-record-carry-skipped-on-restep
    rounds: [1]
    status: resolved
  - key: merge-in-patch-id-admission-unbounded
    rounds: [1]
    status: resolved
  - key: case3-reject-comparison-no-settle-time
    rounds: [1]
    status: resolved
  - key: goto-guard-refusal-promises-amendment-for-merge-review
    rounds: [1]
    status: resolved
  - key: batten-budget-clause-placeholder-run-id
    rounds: [1]
    status: resolved
  - key: goto-refusal-way-forward-omits-resume
    rounds: [1]
    status: resolved
  - key: loom-budget-way-forward-placeholder-signature
    rounds: [1]
    status: resolved
---

Round 1's eight non-gating-or-fixed keys were re-checked in round 2 and none regressed, so they are resolved.
The forced-edit enumeration gap (whole-struct test fixtures and enumerating package docs after a new field or key) was closed at its round-1 sites but recurs at card 7, 15, 13 and 19 targets; it stays open as a BLOCKING design finding.
Round 2 adds one LOW scope finding: card 2's `GenerationStart` skip of the budget arm names no bound on who sets the mark or how often.
