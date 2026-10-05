---
round: 3
ledger:
  - key: endsessionbyname-leaves-socket-file
    rounds: [1]
    status: resolved
  - key: new-go-comments-hard-wrapped
    rounds: [1, 2]
    status: resolved
  - key: comment-rebreaks-split-nonclause-boundaries
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: endsessionbyname-doc-overstates-socket-removal
    rounds: [2]
    status: resolved
  - key: removesocketfileoncegone-elapsed-branch-untested
    rounds: [2]
    status: resolved
  - key: selvage-idle-prompt-ctrlc-test-races-startup
    rounds: [2]
    status: resolved
---

Round 1's socket-file finding stays resolved.
The comment hard-wrapping finding, BLOCKING in round 2, is resolved: the round-3 review finds the range's added comments broken at clause boundaries, apart from two splits.
Those two splits (F1 in `cli_tmux_test.go`, F2 in `ensuresession_integration_test.go`) continue the round-2 key `comment-rebreaks-split-nonclause-boundaries`, and stay open at LOW and NIT-level non-gating severity.
The round-2 doc, coverage and flaky-test findings are resolved: the review confirms the `EndSessionByName` doc matches the code, the elapsed-wait branch is covered, and the Selvage Ctrl-C test passes 25 of 25.
