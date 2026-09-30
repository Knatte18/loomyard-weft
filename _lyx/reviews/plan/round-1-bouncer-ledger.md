---
round: 1
ledger:
  - key: audit-warnings-lost-on-rebegin-and-recovery
    rounds: [1]
    status: resolved
  - key: card8-vacuous-no-duplicate-test
    rounds: [1]
    status: resolved
  - key: batch-warning-to-summary-untested
    rounds: [1]
    status: resolved
  - key: card8-step3-no-report-failure-ungrounded
    rounds: [1]
    status: resolved
  - key: preflight-fabric-sync-and-weft-dirty-way-forward
    rounds: [1]
    status: resolved
  - key: webstercli-fabric-sync-fixture-unreachable
    rounds: [1]
    status: resolved
  - key: loom-commit-records-refusals-missing
    rounds: [1]
    status: resolved
  - key: checkseed-preempted-rows-untestable
    rounds: [1]
    status: resolved
  - key: loom-status-spec-goto-and-budget-reason
    rounds: [1]
    status: resolved
  - key: no-report-policy-finding-skips-verify-rerun
    rounds: [1]
    status: resolved
  - key: card1-lyx-dir-derivation-and-named-spawn-keys
    rounds: [1]
    status: resolved
  - key: card7-reader-claim-and-status-comments
    rounds: [1]
    status: resolved
  - key: unknown-batch-way-forward-in-findbatch
    rounds: [1]
    status: resolved
  - key: card4-recovery-assertion-deferred
    rounds: [1]
    status: resolved
  - key: loom-sweep-card-too-broad
    rounds: [1]
    status: resolved
  - key: card8-failing-verify-test-bound-to-correctness-finding
    rounds: [1]
    status: open
---

Round 1's review raised fifteen findings against the pre-fix plan: one blocking (batch audit warnings lost on re-begin and recovery), the rest fidelity, test-reachability, spec-drift, `ImpactSummary` and granularity issues.
The fixer resolved all fifteen, including splitting the loom sweep into cards 26 (CLI verbs) and 27 (gate rows) and renumbering the doc cards to 28–31.

One new finding came out of the fixer's own edits: inserting the correctness-finding tests into card 8 left the failing-verify bullet saying "the same finding", which now refers to a correctness finding.
That leaves D5's same-call policy-finding verify failure, a must-cover scenario, without an unambiguous test.
It is open going into round 2.
