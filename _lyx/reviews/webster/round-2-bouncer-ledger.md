---
round: 2
ledger:
  - key: endsessionbyname-leaves-socket-file
    rounds: [1]
    status: resolved
  - key: new-go-comments-hard-wrapped
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: comment-rebreaks-split-nonclause-boundaries
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: endsessionbyname-doc-overstates-socket-removal
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: removesocketfileoncegone-elapsed-branch-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: selvage-idle-prompt-ctrlc-test-races-startup
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1's socket-file finding is resolved: the round-2 review found the engine-less removal sound in its core, and left only a doc overstatement and an untested branch, filed as new keys.
Round 1's comment-wrapping finding recurs as F1 at BLOCKING, because the fix covered only part of the range and the moved comments in `cli_tmux_test.go` kept their column wrapping.
F2 is a residue of the same fix: re-breaks at non-clause boundaries.
F3 and F4 are the doc and coverage gaps around `removeSocketFileOnceGone`.
F5 is a flaky `tmux`-tier test that fails the verify gate.
