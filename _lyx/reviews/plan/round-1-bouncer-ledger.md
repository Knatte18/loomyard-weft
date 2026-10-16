---
round: 1
ledger:
  - key: card1-orchestrator-md-inlining-no-bound
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card1-paths-go-comment-outside-target-span
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no prior ledger.
F1 asks card 1's Intent to state the bound on the `ORCHESTRATOR.md` inlining; the review's fix text fixes the wording.
F2 asks card 1's Edit group to swap `internal/orchcli#orchPaths` for the file path `internal/orchcli/paths.go`.
Neither is gating.
