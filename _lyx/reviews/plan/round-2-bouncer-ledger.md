---
round: 2
ledger:
  - key: card5-approve-reject-seed-refusal-invalid-way-forward
    rounds: [1]
    status: resolved
  - key: card5-wirelanding-seed-decode-guard-unreachable
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card5-landingshed-doc-edit-no-anchor
    rounds: [1]
    status: resolved
  - key: card5-runstart-header-stale
    rounds: [1]
    status: resolved
---

Round 1's BLOCKING approve/reject refusal finding and the two LOW doc and header findings are absent from round 2's review and are resolved.
Round 2's F1 continues the `wireLanding` seed-read thread: the branch is reachable through a non-self run id, no test reaches it, and its exemption label is wrong.
It is LOW, scope-class and a local fix, so it does not block.
