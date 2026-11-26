---
round: 1
ledger:
  - key: hub-tmux-server-pattern-misses-clustered-flags-and-spaced-paths
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-guard-rows-untested-in-quoted-scopes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: session-bus-refuses-dbus-send-system
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: summary-misstates-rc-errors-and-blast-radius
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: sampler-stencil-allows-lym-prefixed-trial-server
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger carries forward.
Its findings are all LOW and each states the local correction it asks for.
They concern the two new `shellguard` rows and their tests, the change description, and the sampler stencil.
None is gating.
