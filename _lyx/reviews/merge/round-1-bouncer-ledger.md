---
round: 1
ledger:
  - key: orchestrator-md-673-workaround-survives
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding: the parent moved the #673 `reed add` workaround into `ORCHESTRATOR.md`, and the task removed it from `CLAUDE.md` only.
The fix deletes the `## Workarounds` section of `ORCHESTRATOR.md`.
It is not gating (class `scope`).
