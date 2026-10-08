---
round: 4
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
    status: resolved
  - key: d1-d3-state-depends-on-lyx-at
    rounds: [2]
    status: resolved
  - key: d1-d3-first-seen-no-holder
    rounds: [2]
    status: resolved
  - key: d3-d6-since-restarts-per-call
    rounds: [2]
    status: resolved
  - key: d8-d5-done-at-once-ignores-gate
    rounds: [2]
    status: resolved
  - key: d1-stop-hook-rewrite-drops-turn-end
    rounds: [2]
    status: resolved
  - key: d11-match-field-unnamed
    rounds: [2]
    status: resolved
  - key: d11-later-spawn-reads-switch
    rounds: [2]
    status: resolved
  - key: d1-every-hook-prints-nothing-too-broad
    rounds: [2]
    status: resolved
  - key: d3-interrupt-marker-no-source
    rounds: [2]
    status: resolved
  - key: d3-idle-done-two-since-sources
    rounds: [3]
    status: resolved
  - key: d1-single-write-false-for-large-payloads
    rounds: [3, 4]
    status: open
    class: design
    severity: BLOCKING
  - key: d3-no-precedence-liveness-session-end-signal
    rounds: [3]
    status: resolved
  - key: d1-advisory-reason-wider-than-lyx-at
    rounds: [3]
    status: resolved
  - key: d6-history-bounds-last-change-not-report
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: d3-prompt-offset-not-handed-to-reducer
    rounds: [3]
    status: resolved
  - key: d3-session-end-clear-resume-reads-dead
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-orch-session-id-source
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
  - key: d3-marker-entries-not-same-for-every-reader
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d4-registry-entry-removed-at-exit
    rounds: [4]
    status: open
    class: design
    severity: LOW
  - key: d3-wait-loop-retained-input
    rounds: [4]
    status: open
    class: decision
    severity: LOW
  - key: d1-acceptance-hook-roundtrip-wording
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 and 2 findings stay resolved; no later review re-raised them.
Round 3's `d3-idle-done-two-since-sources`, `d3-no-precedence-liveness-session-end-signal`, `d1-advisory-reason-wider-than-lyx-at` and `d3-prompt-offset-not-handed-to-reducer` were settled in round 4's review and are carried as resolved.

Round 4 raised eight findings, one BLOCKING.
`d1-single-write-false-for-large-payloads` is reopened: round 3 asked for a bound, and the added Bound says the rewrite adds no new way to corrupt a line, which is false because a shell `printf` splits a large payload into several writes and four more hooks now append to the same file.
`d6-history-bounds-last-change-not-report` is reopened as a LOW wording mismatch between D3 and D6 on what bounds a call-time state.
New gating-class: `d3-session-end-clear-resume-reads-dead` (`design`, MEDIUM), since the orch clear cycle ends a session inside a live process.
New non-gating: the orch session-id source, interrupt and api-error entries resting on a transcript read, the registry entry's lifetime at exit, the wait loop's retained input, and the hook round-trip acceptance wording.
