---
round: 2
ledger:
  - key: card9-await-marked-probe-lost-advance
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card9-prebuilt-start-helpers-reshaping
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
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

Round 1 raised thirteen findings; round 2 raises two.
Only card 9's `awaitMarkedProbe` protocol persists: the round 1 fix still leaves a wait for a marked read or probe wait that can never come, and F1 asks for a one-loop rewrite with `seen` taken before the post.
F2 is new: the two helpers that post before `e.start` need no reshaping and must not call `awaitMarkedProbe`.
Every other round 1 key is absent from round 2's findings and is marked resolved.
