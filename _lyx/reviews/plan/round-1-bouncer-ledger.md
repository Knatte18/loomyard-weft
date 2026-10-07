---
round: 1
ledger:
  - key: card12-skips-cli-wiring-regression
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card10-impactsummary-misstates-consumers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card11-skipgit-bound-unstated
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

First round, no prior ledger.
Three findings are open, none gating: the card 12 CLI-wiring regression (F1), the card 10 consumer-count misstatement (F2) and the unstated bound on card 11's `SkipGit` handling (F3).
