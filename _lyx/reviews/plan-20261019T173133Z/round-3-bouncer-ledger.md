---
round: 3
ledger:
  - key: card9-wait-leaves-worker-mid-probe
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card9-stall-subtests-inject-after-start
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card9-pass-bound-and-freshness-wording
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card9-await-marked-probe-lost-advance
    rounds: [1, 2]
    status: resolved
  - key: card9-prebuilt-start-helpers-reshaping
    rounds: [2]
    status: resolved
  - key: card3-root-commit-already-carried-error
    rounds: [1]
    status: resolved
  - key: verify-missing-websterengine
    rounds: [1]
    status: resolved
  - key: card8-session-order-assertion-tmux34
    rounds: [1]
    status: resolved
  - key: card4-gitexec-godocs-stale
    rounds: [1]
    status: resolved
  - key: card3-replay-conflicts-wording-bolt-docs
    rounds: [1]
    status: resolved
  - key: card2-shared-libs-readme-missing-leaf
    rounds: [1]
    status: resolved
  - key: card7-attach-check-placement
    rounds: [1]
    status: resolved
  - key: card3-index-differs-return-reachable
    rounds: [1]
    status: resolved
  - key: card4-git-probe-bypasses-exec-core
    rounds: [1]
    status: resolved
  - key: card1-mode-assertion-windows
    rounds: [1]
    status: resolved
  - key: card6-hook-bypass-unbounded
    rounds: [1]
    status: resolved
  - key: card2-audit-doc-ahead-of-cards
    rounds: [1]
    status: resolved
---

Round 1 raised thirteen findings, round 2 two, round 3 three.
Every finding in round 3 is on card 9's `awaitMarkedProbe` and `awaitTyped` helpers.
The round 2 liveness concern is rejected in round 3 and resolved.
Round 3's F1 is the next defect in the same area: the helpers return with the worker mid-probe, so the tick steps after them race it.
F2 covers the stall subtests that inject after `e.start`, and F3 is a wording correction to the pass bound and the freshness sentence.
All other keys from rounds 1 and 2 stay resolved.
