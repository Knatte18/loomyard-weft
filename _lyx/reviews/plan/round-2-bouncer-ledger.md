---
round: 2
ledger:
  - key: card23-misses-battencli-step-tests
    rounds: [1]
    status: resolved
  - key: card19-misses-orch-refusalcase
    rounds: [1]
    status: resolved
  - key: promote-start-guard-misses-stop-after-step2
    rounds: [1]
    status: resolved
  - key: card9-order-beam-refusal-unreached
    rounds: [1]
    status: resolved
  - key: card23-extends-rewritten-cli-file
    rounds: [1]
    status: resolved
  - key: card24-beam-reed-session-unbounded
    rounds: [1]
    status: resolved
  - key: missing-origin-record-refusal-wrong-way-forward
    rounds: [1]
    status: resolved
  - key: adopt-local-records-branch-refusal-no-way-forward
    rounds: [1]
    status: resolved
  - key: card24-window-help-text-stale
    rounds: [1]
    status: resolved
  - key: promote-root-pair-dead-end
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card4-parent-refusals-weak-way-forward
    rounds: [1]
    status: resolved
  - key: slug-rule-suggestion-unchecked
    rounds: [1]
    status: resolved
  - key: beam-false-refusal-names-wrong-verbs
    rounds: [1]
    status: resolved
  - key: card23-self-pair-state-misstated
    rounds: [1]
    status: resolved
  - key: card34-cliwire-constraint-silent
    rounds: [1]
    status: resolved
  - key: missing-records-worktree-branch-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised fifteen findings, none BLOCKING; two were gating `design` findings.
Round 2 raised two MEDIUM findings and no `design` finding, so every round 1 key not raised again is resolved.
`promote-root-pair-dead-end` is reopened as F1: the root-pair refusals of cards 35 and 8 name the cause but no way forward, and the fix is Intent and test prose only.
`missing-records-worktree-branch-untested` is new as F2: the missing-records-worktree branch of the missing-origin-record refusal has no reaching test row in cards 2 and 8.
