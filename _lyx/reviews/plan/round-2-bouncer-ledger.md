---
round: 2
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
    rounds: [1, 2]
    status: resolved
  - key: classify-violation-not-link-aware
    rounds: [2]
    status: resolved
  - key: later-card-drift-bypasses-d5-recording
    rounds: [2]
    status: resolved
  - key: card27-done-state-seed-pointer
    rounds: [2]
    status: resolved
  - key: card23-cites-card-range-not-shared-decision
    rounds: [2]
    status: resolved
---

In round 1 the fixer resolved all fifteen findings, but its test insertions left card 8's failing-verify bullet bound to a correctness finding.
That item stayed open into round 2.

Round 2's review raised it again as F1 and added four new findings.
Two were MEDIUM: `ClassifyViolation`'s `_lyx` prefix test was lexical, so it missed a write spelled through the weft link.
Later-card drift warnings also reached only the envelope, never state.json or the summary.
Two were NITs: card 27's done-state pointer, and card 23's card-range citation.
The fixer resolved all five.
Card 8 and card 11 now name the same-call policy-finding-with-failing-verify case, card 1 compares link-resolved paths and tests a link-spelled write, and card 10 records drift through `recordBatchWarning` with card 2's helpers taking class and detail strings.
Card 14's summary explanation covers drift.

The fixer's edits introduced no new finding, and every entry is resolved.
