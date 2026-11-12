---
round: 1
ledger:
  - key: detached-pass-output-pipe-sigpipe
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: anchor-move-spares-testmap-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: orchcli-stale-refresh-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: landings-since-test-map-readability-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: reconcile-walk-test-map-every-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 is the first ledger.
F1 maps to `detached-pass-output-pipe-sigpipe`, the only gating finding: the pass's output must go somewhere that outlives the dobby.
F2 to F5 map to the other four keys, each a missing test or stale wording with the fix text already settled.
