---
round: 1
ledger:
  - key: gitexec-doc-stall-bound-splits-pause-hold
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 only: the `# Stall bound` section in `internal/gitexec/doc.go` sits between the pause-hold sentences, so "The hold only delays the call's start and never fails it." lost its referent.
The fix is a local sentence move whose result the finding text fixes.
