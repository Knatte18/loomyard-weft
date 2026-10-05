---
round: 6
ledger:
  - key: escalation-record-unreadable-refusal-unspecified
    rounds: [1]
    status: resolved
  - key: loom-review-key-refusal-no-way-forward
    rounds: [2]
    status: resolved
  - key: decision-add-commit-failure-no-spec-row
    rounds: [3]
    status: resolved
  - key: decision-add-no-decisions-heading-no-way-forward
    rounds: [3]
    status: resolved
  - key: bouncer-circling-decision-refusals-no-way-forward
    rounds: [3]
    status: resolved
  - key: circling-verb-malformed-escalation-no-way-forward
    rounds: [4]
    status: resolved
  - key: decision-add-status-read-and-append-no-retry-clause
    rounds: [4]
    status: resolved
  - key: escalation-unreadable-reason-lacks-way-forward-label
    rounds: [4]
    status: resolved
  - key: config-review-key-spec-cell-disagrees-with-message
    rounds: [4]
    status: resolved
  - key: circling-verb-helper-error-refusals-no-way-forward
    rounds: [5]
    status: resolved
  - key: decision-add-retry-refusals-no-spec-row
    rounds: [5]
    status: resolved
  - key: loomcli-touched-files-not-gofmt-clean
    rounds: [5]
    status: resolved
  - key: escalation-write-failure-no-way-forward-row-or-test
    rounds: [6]
    status: open
  - key: unreadable-rows-understate-trigger
    rounds: [6]
    status: open
---

Rounds 1 to 5 each found further sites of `PATTERN-refusal-way-forward`.
The round-6 review swept the circling and decision verbs and the shedadapters Reasons and found the round-5 sites compliant, so those three keys are resolved.
Round 6 raised F1, a failed escalation write that halts the run with no way forward, spec row or test, and F2, spec rows that understate when the unreadable Reason fires.
Both are `consistency`, so neither gates, and both stay open until a fresh review sees their fixes.
