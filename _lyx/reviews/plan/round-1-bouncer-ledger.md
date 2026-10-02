---
round: 1
ledger:
  - key: interim-gate-attempts-mapping-and-planwrite-assertion
    rounds: [1]
    status: resolved
  - key: per-entry-counts-unsized-on-attach-path
    rounds: [1]
    status: resolved
  - key: removed-gate-key-comments-outside-card-2
    rounds: [1]
    status: resolved
  - key: card-1-doc-comment-sweep-misses-engine-and-singlellm
    rounds: [1]
    status: resolved
  - key: impactsummary-lacks-concrete-blast-radius
    rounds: [1]
    status: resolved
  - key: card-2-overstates-entry-qualified-resolver-errors
    rounds: [1]
    status: resolved
  - key: no-action-constraints-unrecorded
    rounds: [1]
    status: resolved
---

This is the first round, with no prior ledger.
The round-1 review raised seven findings: two MEDIUM (F1, the interim `gate_attempts` mapping, and F2, the per-entry count sizing on the attach path), four LOW (F3–F6, covering comment coverage, ImpactSummary concreteness and error qualification), and one NIT (F7, the unrecorded no-action constraints).
The fixer addressed all seven in the cards, and the judge verified each fix against the current plan text and the tree.
All seven are resolved and none is open.
