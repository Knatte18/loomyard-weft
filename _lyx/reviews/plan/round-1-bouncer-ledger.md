---
round: 1
ledger:
  - key: card6-misses-tests-broken-by-follow-pin
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card2-failed-client-listing-and-stale-attachargv-doc
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card3-realtmux-later-attaches-not-waited-for
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-live-fallback-row-tests-unreachable-state
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings against the follow plan.
Card 6's `@lyx_follow_command` pin raises the exact `set-option` counts in three tests the card does not list.
Card 2 leaves the failed client listing undecided and `AttachArgv`'s doc comment stale.
Card 3's real-tmux test races the later client attaches.
Card 6's live fallback row unsets the option under a binding pinned with the jump, a state no pin produces.
All four fixes are local corrections.
