---
round: 1
ledger:
  - key: inflight-inject-retyped-after-crash
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: program-sender-vs-ancestry-precedence
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: linux-ancestry-escapable-by-detach
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: inject-ttl-not-restart-derivable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reconnecting-waiter-misses-outcome
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: removed-holder-queued-injects-unresolved
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: interrupt-passes-hold
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: several-holders-one-receiver
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: decision-behaviors-lack-criteria
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: support-log-q8-stale-ttl
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orch-self-inject-two-rules
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: numbered-orch-role-undecided
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: false-note-no-socket-dial
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-cross-hub-clause-out-of-scope
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised fourteen findings and no earlier ledger exists, so each key is new.
The blocking pair concerns inject delivery across a crash and sender identity precedence.
The remaining gating design findings concern the ancestry bypass, TTL persistence, waiter reconnect, removed-holder injects and interrupt under a hold.
The mechanical findings are local edits (criteria, a stale ledger entry, a note, a clause).
