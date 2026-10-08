---
round: 3
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
    status: open
    class: decision
    severity: MEDIUM
  - key: d1-single-write-false-for-large-payloads
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-no-precedence-liveness-session-end-signal
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d1-advisory-reason-wider-than-lyx-at
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: d6-history-bounds-last-change-not-report
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: d3-prompt-offset-not-handed-to-reducer
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised twelve findings and round 2 raised ten; the round-3 review did not re-raise any of them, so all are carried as resolved.
Round 2's findings on the stamps, first-seen time, per-call restart and the Stop hook rewrite were fixed; round 3 found new defects in the fixes' neighbourhood and no recurring key.

Round 3 raised six findings, none BLOCKING.
Gating: `d1-single-write-false-for-large-payloads` (`design`, MEDIUM), because the shell `printf` splits large Stop payloads into several writes and the Decision states no bound for a stamped but corrupt line.
Non-gating MEDIUM: `d3-idle-done-two-since-sources` (where the output-file fact is applied) and `d3-no-precedence-liveness-session-end-signal` (no ordering among liveness, session end, newest signal).
LOW: the advisory reason wider than `lyx_at`, the history bounding the last change and not the last report, and the prompt offset missing from the reducer's inputs.
