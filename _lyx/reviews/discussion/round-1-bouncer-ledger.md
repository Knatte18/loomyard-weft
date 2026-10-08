---
round: 1
ledger:
  - key: mid-merge-prime-stale-config-inherited
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: preflight-stop-missing-from-decision-9-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: decision-14-role-membership-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: hub-lock-wait-unbounded-reconcile
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: relocation-decision-14-rationale-and-open-risks
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first ledger.
The gating item is the mid-merge prime case: Decision 7's skip lets `lyx batten run` fork a stale-config pair, which contradicts Decision 4 and its acceptance criterion.
The two MEDIUM consistency items concern Decision 9's bound missing `Preflight`'s stop and Decision 14's role test contradicting its exclusions.
The LOW design item concerns the unbounded per-pair commit wait under the hub lock and `Preflight`'s unstated lock-timeout outcome.
The NIT proposes cutting Decision 14's rationale and two restating Open risks.
