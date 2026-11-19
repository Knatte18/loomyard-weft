---
round: 1
ledger:
  - key: regression-test-bypasses-verb
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: sampler-watch-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: test-builds-pair-per-run
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all class consistency.
F1 (BLOCKING) asks the pr-findings regression test to drive the verb through `clihelp.ExecuteIn` instead of calling the edge directly.
F2 (LOW) asks for the sampler-watch comment in `wireLanding` to name the send-back edge.
F3 (LOW) asks the test to copy the pair-carrying hub shape instead of calling `AddPair`.
Each fix text fixes the result, so all are mechanical.
