---
round: 4
ledger:
  - key: d12-commit-test-inherited-startsha
    rounds: [1]
    status: resolved
  - key: d7-allowlist-refuses-nested-orch
    rounds: [1, 2, 3]
    status: resolved
  - key: d8-reader-write-flags-unbounded
    rounds: [1, 2, 3, 4]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-recoveries-count-not-carried
    rounds: [1, 2, 3]
    status: resolved
  - key: d12-new-refusal-unmarked
    rounds: [1]
    status: resolved
  - key: d9-gated-leftover-files
    rounds: [1, 2, 3]
    status: resolved
  - key: d1-warning-no-sink-restamp-save
    rounds: [1]
    status: resolved
  - key: d1-d2-refusals-uncovered
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: friction-constraint-no-plumbing
    rounds: [1]
    status: resolved
  - key: goal-stale-tally-interview-contradiction
    rounds: [1]
    status: resolved
  - key: scope-bullets-without-criteria
    rounds: [1, 2, 3]
    status: resolved
  - key: mergeresolve-import-question-in-support-log
    rounds: [1]
    status: resolved
  - key: d11-candidates-perishable-criterion-not-red
    rounds: [1]
    status: resolved
  - key: d8-d5-splitter-and-pinned-tests
    rounds: [1, 2]
    status: resolved
  - key: d7-guard-refuses-detached-push
    rounds: [2]
    status: resolved
  - key: d7-refusal-spec-ways-forward
    rounds: [2, 3, 4]
    status: open
    class: decision
    severity: LOW
  - key: d12-card-amended-cap-no-master-rung
    rounds: [2, 3]
    status: resolved
  - key: d9-test-claims-disagree
    rounds: [2, 3]
    status: resolved
  - key: d8-classifier-seam-unstated
    rounds: [2, 3]
    status: resolved
  - key: d7-bypass-bound-topology-verbs
    rounds: [3]
    status: resolved
  - key: d7-gate-inherited-strand-name-tests
    rounds: [3, 4]
    status: open
    class: design
    severity: LOW
  - key: d3-log-in-shared-helper
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: d1-stuck-mapping-identification
    rounds: [3]
    status: resolved
  - key: d11-push-candidate-contradicts-test
    rounds: [3]
    status: resolved
  - key: d7-d8-reader-lists-differ
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: d8-test-row-and-ledger-slips
    rounds: [3]
    status: resolved
  - key: d8-signature-shape-pinned
    rounds: [3]
    status: resolved
  - key: d12-begin-batch-dead-refusal-stale
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d1-save-failure-after-rebaseline-unplaced
    rounds: [4]
    status: open
    class: decision
    severity: LOW
  - key: d12-recovery-retry-read-failure
    rounds: [4]
    status: open
    class: decision
    severity: LOW
  - key: d9-completion-signal-constraint-missing
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: d7-rationale-false-premise
    rounds: [4]
    status: open
    class: design
    severity: LOW
  - key: d12-recoveries-definition-counted-spawns
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised fourteen findings, round 2 nineteen, round 3 eighteen, round 4 twelve; each round carries one BLOCKING.

The D8 read-only classifier is the recurring centre and the only gating key left.
Round 1 found unbounded flags, round 2 bounded dash-words, round 3 found positional words, and round 4 finds quoted, escaped, `$VAR` and brace-expanded words that the exact-flag check does not see.
The pattern is a deny-list of shapes growing one instance per round; a rule that every word of a `git` form is plain would close the class.

Round 3's other MEDIUM keys (D7 allowlist and bypass bound, D12 counting and card-amended rung, D9 claims and leftover files, D8 seam, scope bullets) were not re-raised, and the review's focus verdicts call D7's guard and D12's counting settled, so they are resolved.
Round 4 adds one MEDIUM, a stale begin-batch dead refusal under D12, and LOW or NIT items on D1 save placement, D7 rationale and refusal step, D9 and D8 constraints, and recovery_retry read failures.
