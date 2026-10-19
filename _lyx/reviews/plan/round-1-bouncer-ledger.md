---
round: 1
ledger:
  - key: card9-await-marked-probe-lost-advance
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card3-root-commit-already-carried-error
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: verify-missing-websterengine
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card8-session-order-assertion-tmux34
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card4-gitexec-godocs-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-replay-conflicts-wording-bolt-docs
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card2-shared-libs-readme-missing-leaf
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card7-attach-check-placement
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-index-differs-return-reachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-git-probe-bypasses-exec-core
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: card1-mode-assertion-windows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-hook-bypass-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card2-audit-doc-ahead-of-cards
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review; the ledger has no earlier entries.
The gating items are F1 (card 9's `awaitMarkedProbe` can lose the clock advance) and F3 (`./internal/websterengine` missing from both verify lines).
F2 changes card 3's result for an already-carried root commit and needs a fresh look.
The rest are local text corrections.
