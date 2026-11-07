---
round: 2
ledger:
  - key: openlaunch-effort-untested
    rounds: [1]
    status: resolved
  - key: loadconfig-agents-yaml-untested
    rounds: [1]
    status: resolved
  - key: orch-reload-list-wiring-untested
    rounds: [1]
    status: resolved
  - key: configrole-not-asserted
    rounds: [1]
    status: resolved
  - key: effort-not-validated-at-load
    rounds: [1]
    status: resolved
  - key: config-help-line-removed
    rounds: [1]
    status: resolved
  - key: unknown-effort-key-not-pinned
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: openlaunch-doc-omits-effort-default
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings; round 2 does not re-raise any and its settled points show each addressed.
Round 2 raised two LOW consistency findings: the unpinned key name in the unknown-effort refusal test, and the missing effort-default sentence in the `OpenLaunch` doc comment.
Neither gates.
