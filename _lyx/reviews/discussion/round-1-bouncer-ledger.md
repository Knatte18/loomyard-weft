---
round: 1
ledger:
  - key: d4-seed-no-git-route
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d6-apply-drops-unmerged-edit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-failed-commit-and-precommit-validation
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: seed-source-report-only-under-risks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d4-template-fallback-and-prime-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: doc-rewrite-detail-under-auto-mode
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

First round, no prior ledger.
Six findings are open, all about `decision-record.md`.
Two are gating design findings: the D6 `--apply` deletion that can lose an unmerged per-worktree edit, and the D4 template fallback with its unverified prime premise.
Two decision findings concern the missing commit route for the D4 seed and the D5 failed-commit and pre-commit validation gaps.
Two LOW consistency findings concern content sitting in the wrong section: the seed-source report under Open risks, and the `doc.go` rewrite detail under Auto-mode assumptions.
