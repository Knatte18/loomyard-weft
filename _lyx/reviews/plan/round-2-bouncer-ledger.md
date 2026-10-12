---
round: 2
ledger:
  - key: card7-unseen-clear-hook-breaks-untargeted-attach-tests
    rounds: [1]
    status: resolved
  - key: card1-breaks-attach-plan-error-row
    rounds: [1]
    status: resolved
  - key: card4-relaunch-kill-unlicensed-by-d6-stale-listing
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
  - key: card4-empty-new-window-answer-guard-untested
    rounds: [1]
    status: resolved
  - key: card12-orelse-fallback-states-no-bound
    rounds: [1]
    status: resolved
  - key: comment-rewrites-outside-card-targets
    rounds: [1]
    status: resolved
  - key: overview-md-second-shuttle-verb-list-stale
    rounds: [1]
    status: resolved
---

Round 1 raised seven findings; the BLOCKING card 7 finding and the design finding on card 4's relaunch kill gated.
Round 2 raised one finding, F1, on the same card 4 relaunch kill, now classed `decision` because the decision record forbids the kill (D5, D6).
That key stays open at MEDIUM and is not gating-class.
The other six round-1 keys did not recur and are resolved.
