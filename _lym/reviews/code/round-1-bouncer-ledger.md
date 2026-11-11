---
round: 1
ledger:
  - key: waiting-post-liveness-test-row-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: cli-waiting-post-not-live-test-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: receiver-live-lost-on-waiting-post-reconnect
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: callwatching-doc-names-only-inject
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: creelcli-doc-warning-line-placement
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: post-help-omits-receiver-live-and-warning
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round and has no earlier ledger.
Six findings, none gating: four LOW (F1 to F4) and two NIT (F5, F6).
F3 is the only `design` finding, at LOW, so the loop converges.
