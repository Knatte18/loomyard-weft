---
round: 2
ledger:
  - key: card6-misses-tests-broken-by-follow-pin
    rounds: [1]
    status: resolved
  - key: card2-failed-client-listing-and-stale-attachargv-doc
    rounds: [1, 2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card3-realtmux-later-attaches-not-waited-for
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card6-live-fallback-row-tests-unreachable-state
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card2-warn-mismatched-clients-doc-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-weird-named-session-click-lacks-status-line
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised four findings; the card 6 test-count finding did not return and is resolved.
Round 2 re-raises the card 2 failed-listing area as F1 (the fix left `TestAttachArgv_MultiClientWarning` broken), the card 3 real-tmux client counts as F5, and the card 6 fallback row as F4 (no row reaches `pinBindingsLocked`'s choice).
F2 (stale `warnMismatchedClientsLocked` doc comment) and F3 (weird-named session lacks the second status line) are new.
All fixes in the review are local corrections.
