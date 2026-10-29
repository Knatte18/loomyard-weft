---
round: 1
ledger:
  - key: d10-wait-marker-kind-unnamed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d9-merged-lands-ungated-head
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-session-limit-unenforced
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: operator-cannot-approve-own-pr
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: d11-github-merge-skips-fabric-merge-records
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-session-identifiers-no-home
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-merged-session-live-dirty-tree
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d11-disallowed-merge-method-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d11-no-leaves-pr-to-teardown-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-resume-contradicts-d7-no-restart
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-way-forward-cancel-then-resume
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d12-teardown-gone-worktree
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d9-followup-not-named-board-note
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised thirteen findings and no earlier ledger exists.
The BLOCKING cluster is the pull-request row's bounds: which wait marker keeps the quiet check silent (F1), which head a merge on GitHub lands (F2), and what enforces the session's no-push limit (F3).
F4 is a parent question about how the operator approves a pull request opened under their own account.
The MEDIUM findings concern what replaces `Fabric.Merge`'s records-side work (F5), where the session identifiers live (F6), the merged-state read sequence (F7) and the disallowed-merge-method halt (F8).
The LOW findings are wording and consistency corrections across D3, D4, D7, D9, D11 and D12.
