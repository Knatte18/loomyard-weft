---
round: 1
ledger:
  - key: hard-cap-check-precedes-escalation-preconditions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: capped-decide-fragment-handed-on-no-verb
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised two non-gating consistency findings.
F1 (`hard-cap-check-precedes-escalation-preconditions`): `circlingVerb` refuses on the hard cap before the not-escalated and already-decided checks.
F2 (`capped-decide-fragment-handed-on-no-verb`): the capped decide fragment mentions a hand-on that names no verb.
