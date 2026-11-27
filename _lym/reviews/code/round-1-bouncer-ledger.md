---
round: 1
ledger:
  - key: subagent-ttl-scope-understated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: main-ttl-doc-reads-as-all-requests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: testopenlaunch-doc-omits-subagent-export
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 found the diff makes the change the board entry asks for.
Its three findings are about wording:
the prose understates which requests the subagent TTL variable covers (F1),
the main-conversation TTL variable's doc claims too wide a reach (F2),
and the `TestOpenLaunch` doc comment omits an export the test pins (F3).
