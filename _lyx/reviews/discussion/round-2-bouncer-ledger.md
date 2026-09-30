---
round: 2
ledger:
  - key: run-exit-cross-check-ignores-dispositions
    rounds: [1]
    status: resolved
  - key: parent-write-split-overlaps-run-state
    rounds: [1]
    status: resolved
  - key: goto-budget-reset-and-jump-entry-undecided
    rounds: [1]
    status: resolved
  - key: invariant-unscoped-violated-on-landing
    rounds: [1]
    status: resolved
  - key: goto-reentry-gate-deferred
    rounds: [1]
    status: resolved
  - key: consumed-correctness-parent-write-unaudited
    rounds: [1]
    status: resolved
  - key: rebaseline-misses-regrouped-batches
    rounds: [1]
    status: resolved
  - key: verify-cannot-run-overlaps-io-row
    rounds: [1]
    status: resolved
  - key: goto-running-reads-as-crash
    rounds: [1]
    status: resolved
  - key: per-row-test-criterion-class-mismatch
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: resolved
  - key: run-exit-undispositioned-findings-no-disposition
    rounds: [1, 2]
    status: resolved
  - key: d5-recording-path-batch-scoped
    rounds: [2]
    status: resolved
  - key: roadmap-question-deferred-to-plan
    rounds: [2]
    status: resolved
  - key: refusal-kind-rule-restated-with-tally
    rounds: [2]
    status: resolved
---

# Cross-round ledger

Round 1 raised eleven findings, and all were fixed that round.
It left one gap open: a run-exit finding no record-batch ever saw had no disposition.

Round 2's report raised that gap again as its BLOCKING F1, plus three smaller findings:
D5's batch-only recording path (F2), a roadmap question deferred into the support log (F3), and a refusal-kind rule stated three times with a stale tally (F4).
The artifacts as judged fix all four.
D8 dispositions run-exit-only findings by class, D5 has a run-level warning home, ledger item 18 is resolved, and the rule sits once in Constraints.

Every ledger entry is now resolved, and the discussion is approved at round 2.
One stale support-log wording (ledger item 19's "hard-errors" answer, superseded by item 22) was observed but not raised, since `Plan-Write` never reads the support log and D8 is unambiguous.
