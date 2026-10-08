---
round: 2
ledger:
  - key: d8-immediate-done-vs-d3-d5-mapping
    rounds: [1]
    status: resolved
  - key: d3-call-time-inputs-unsourced
    rounds: [1]
    status: resolved
  - key: support-log-same-history-claim
    rounds: [1]
    status: resolved
  - key: d11-python-deny-let-through-unstated
    rounds: [1]
    status: resolved
  - key: d8-hung-shell-visibility-and-timeout
    rounds: [1]
    status: resolved
  - key: d1-hooks-no-stated-bound
    rounds: [1]
    status: resolved
  - key: d6-dead-state-unreachable
    rounds: [1]
    status: resolved
  - key: goal-part3-overstates-d8
    rounds: [1]
    status: resolved
  - key: byte-identical-denies-claim
    rounds: [1]
    status: resolved
  - key: decisions-without-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: rationale-narrative-belongs-in-support-log
    rounds: [1]
    status: resolved
  - key: q14-payload-set-mismatch
    rounds: [1]
    status: resolved
  - key: d3-since-stamp-undefined-for-states
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-d3-state-depends-on-lyx-at
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d1-d3-first-seen-no-holder
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-d6-since-restarts-per-call
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-d5-done-at-once-ignores-gate
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d1-stop-hook-rewrite-drops-turn-end
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-match-field-unnamed
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d11-later-spawn-reads-switch
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d1-every-hook-prints-nothing-too-broad
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d3-interrupt-marker-no-source
    rounds: [2]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised twelve findings, none BLOCKING, and the round-2 review found none of them regressed, so all twelve are carried as resolved.
Round 2 raised ten new findings, none BLOCKING and none recurring by key.
Several refine round-1 fixes: the per-state `since` stamps, the per-reader history and the gated-run condition.

Gating at MEDIUM `design`: the undefined `since` stamps for several states, the missing holder of first-seen time, `since` restarting on every call for call-time states, and the Stop hook rewrite that can drop a turn end.
Non-gating MEDIUM: the state depending on the agent-writable `lyx_at`, D8 and D5 ignoring the gate, and D11 not naming the field its match reads.
LOW: the later-spawn read of the python switch, the over-broad "prints nothing" sentence, and the unsourced interrupt marker.
