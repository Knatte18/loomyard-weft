---
round: 2
ledger:
  - key: d6-reprompt-retry-never-fires
    rounds: [1]
    status: resolved
  - key: stopped-outside-file-contract-first
    rounds: [1]
    status: resolved
  - key: d1-unmatched-turn-start-holds-idle-pane
    rounds: [1]
    status: resolved
  - key: d4-no-work-done-premise-unbounded
    rounds: [1]
    status: resolved
  - key: d2-settle-wait-vs-d3-window
    rounds: [1]
    status: resolved
  - key: stopped-overwrite-vs-burler-criterion
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
  - key: stop-verb-record-write-failure
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: resolved
  - key: q8-open-while-d4-decided
    rounds: [1]
    status: resolved
  - key: restated-content-in-decision-record
    rounds: [1]
    status: resolved
  - key: failed-send-leaves-text-blocks-retry
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d6-unmatched-turn-start-no-override
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-overshoot-bound-undefined-appearance-check
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: completion-signal-tripwire-misses-stopped-write
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: constraint-contradicts-scope-on-attach
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d1-turn-start-interrupt-capability-misplaced
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d8-text-not-scoped-to-outside-shuttle
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's ten findings were fixed; the round-2 reviewer found none of them again in their round-1 form.
`stopped-overwrite-vs-burler-criterion` continues as round 2's F4: the burler criterion still misses a half stopped by guid during `join` while its `Wait` runs.

Round 2 raised eight findings.
The BLOCKING one is that a not-landed send leaves its text in the box, so D6's retry never re-sends (F1).
Three more `design` findings are open at MEDIUM: the missing override for an unmatched `turn_start` at the gate boundary (F2), the undefined appearance check behind the overshoot bound (F3), and the completion-signal tripwire not seeing the `stopped` write (F5).
Three LOW `consistency` findings cover the attach Constraint, D1's prober capability and D8's scope wording (F6, F7, F8).
