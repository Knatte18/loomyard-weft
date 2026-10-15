---
round: 2
ledger:
  - key: inflight-inject-retyped-after-crash
    rounds: [1]
    status: resolved
  - key: program-sender-vs-ancestry-precedence
    rounds: [1]
    status: resolved
  - key: linux-ancestry-escapable-by-detach
    rounds: [1]
    status: resolved
  - key: inject-ttl-not-restart-derivable
    rounds: [1]
    status: resolved
  - key: reconnecting-waiter-misses-outcome
    rounds: [1]
    status: resolved
  - key: removed-holder-queued-injects-unresolved
    rounds: [1]
    status: resolved
  - key: interrupt-passes-hold
    rounds: [1]
    status: resolved
  - key: several-holders-one-receiver
    rounds: [1]
    status: resolved
  - key: decision-behaviors-lack-criteria
    rounds: [1]
    status: resolved
  - key: support-log-q8-stale-ttl
    rounds: [1]
    status: resolved
  - key: orch-self-inject-two-rules
    rounds: [1]
    status: resolved
  - key: numbered-orch-role-undecided
    rounds: [1]
    status: resolved
  - key: false-note-no-socket-dial
    rounds: [1]
    status: resolved
  - key: d3-cross-hub-clause-out-of-scope
    rounds: [1]
    status: resolved
  - key: sender-identity-fails-open-on-unreadable-state
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: waiting-verbs-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: inject-ttl-under-other-senders-hold
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: hold-on-orch-in-stage-1
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: off-linux-bound-omits-in-pane-claims
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pane-process-not-in-persisted-reed-state
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: program-sender-shortname-unchecked
    rounds: [2]
    status: open
    class: decision
    severity: NIT
    fix: mechanical
  - key: event-and-role-names-drift
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

All fourteen round 1 keys are resolved: none is raised again in round 2, and the review's Rejected section clears the crash-retype case.
Round 2 raised eight new keys and no BLOCKING finding.
The two MEDIUM design findings concern a fail-open sender identity when strand state is unreadable and unbounded `--wait` verbs.
The LOW and NIT findings are decisions left unstated, one design source error in D1, and wording drift.
