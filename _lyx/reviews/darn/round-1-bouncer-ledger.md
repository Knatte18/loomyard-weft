---
round: 1
ledger:
  - key: summary-pack-file-departure-untraced
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: fragment-banners-name-fork-prompt
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two non-gating findings.
The summary does not state that the separate pack file departs from the board entry or where the decision is recorded.
Three pack fragment banners still say the fork prompt although they fill the pack file.
Both fixes are local text corrections.
