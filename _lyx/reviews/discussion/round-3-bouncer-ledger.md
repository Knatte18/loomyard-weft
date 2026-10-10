---
round: 3
ledger:
  - key: d12-commit-test-inherited-startsha
    rounds: [1]
    status: resolved
  - key: d7-allowlist-refuses-nested-orch
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d8-reader-write-flags-unbounded
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-recoveries-count-not-carried
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d12-new-refusal-unmarked
    rounds: [1]
    status: resolved
  - key: d9-gated-leftover-files
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: LOW
  - key: d1-warning-no-sink-restamp-save
    rounds: [1]
    status: resolved
  - key: d1-d2-refusals-uncovered
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: friction-constraint-no-plumbing
    rounds: [1]
    status: resolved
  - key: goal-stale-tally-interview-contradiction
    rounds: [1]
    status: resolved
  - key: scope-bullets-without-criteria
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: LOW
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
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d12-card-amended-cap-no-master-rung
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d9-test-claims-disagree
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d8-classifier-seam-unstated
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d7-bypass-bound-topology-verbs
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-gate-inherited-strand-name-tests
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d3-log-in-shared-helper
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: d1-stuck-mapping-identification
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: d11-push-candidate-contradicts-test
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: d7-d8-reader-lists-differ
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: d8-test-row-and-ledger-slips
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
  - key: d8-signature-shape-pinned
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised fourteen findings, round 2 nineteen, round 3 eighteen; each round carries one BLOCKING.

The D8 read-only classifier is the recurring centre: flags were unbounded in round 1, a bound on dash-words was added in round 2, and round 3 shows positional words (`git remote -v add`, `git branch name`) still write, now BLOCKING.
D7's guard drew a new BLOCKING in round 2 (detached push), resolved in round 3; round 3 instead finds its bypass bound, refusal-spec rows, gate-inherited strand name and nested-sync discriminator wanting.
D12 stays open on the counting seam and now on Master's other dead rungs.
The round 2 splitter key and the detached-push key were not re-raised and are resolved.
Round 3 LOW and NIT items are tracked under new keys.
