---
round: 1
ledger:
  - key: d8-immediate-done-vs-d3-d5-mapping
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d3-call-time-inputs-unsourced
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: support-log-same-history-claim
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d11-python-deny-let-through-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-hung-shell-visibility-and-timeout
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-hooks-no-stated-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d6-dead-state-unreachable
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: goal-part3-overstates-d8
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: byte-identical-denies-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: decisions-without-acceptance-criterion
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: rationale-narrative-belongs-in-support-log
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: q14-payload-set-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first review of the discussion pair.
Twelve findings, none BLOCKING.
Gating: the D3 call-time input list, the D11 python deny let-through side, and the D8 hung-shell visibility and Timeout outcome.
The D8/D3/D5 state-mapping mismatch and the support-log "same history" claim are MEDIUM `consistency` and get fixed but do not gate.
The remainder are LOW or NIT bounds, wording and relocation items.
