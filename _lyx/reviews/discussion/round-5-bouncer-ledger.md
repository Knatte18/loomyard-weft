---
round: 5
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
    status: resolved
  - key: d3-no-precedence-liveness-session-end-signal
    rounds: [3]
    status: resolved
  - key: d1-advisory-reason-wider-than-lyx-at
    rounds: [3]
    status: resolved
  - key: d6-history-bounds-last-change-not-report
    rounds: [3, 4]
    status: resolved
  - key: d3-prompt-offset-not-handed-to-reducer
    rounds: [3]
    status: resolved
  - key: d3-session-end-clear-resume-reads-dead
    rounds: [4]
    status: resolved
  - key: d3-orch-session-id-source
    rounds: [4]
    status: resolved
  - key: d3-marker-entries-not-same-for-every-reader
    rounds: [4]
    status: resolved
  - key: d4-registry-entry-removed-at-exit
    rounds: [4]
    status: resolved
  - key: d3-wait-loop-retained-input
    rounds: [4]
    status: resolved
  - key: d1-acceptance-hook-roundtrip-wording
    rounds: [4]
    status: resolved
  - key: d1-events-mtime-moves-batten-activity
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-d3-stamp-payload-split-across-polls
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-waiting-turn-end-transcript-fallback-not-signal-only
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-precedence-contradicts-output-files-first
    rounds: [5]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d3-newest-turn-start-per-reader-after-interrupt
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
  - key: d1-rationale-chunking-measurement-belongs-in-support-log
    rounds: [5]
    status: open
    class: scope
    severity: NIT
---

Rounds 1 to 3 findings stay resolved.
Round 4's eight findings are carried as resolved: round 5's review judged the large-payload write bound, the bounding history entry, the `SessionEnd` `clear` or `resume` case and the orch session id as not defects, and it did not re-raise the registry entry lifetime, the wait loop's retained input or the hook round-trip wording.
The round-4 marker-entry finding is resolved too; round 5's F3 and F5 are new edge cases of signal-only entries and are keyed separately.

Round 5 raised six findings, none BLOCKING.
Gating-class: `d1-events-mtime-moves-batten-activity` (batten's quiet clock reads the events file mtime), `d1-d3-stamp-payload-split-across-polls` (a poll between stamp and payload gives an empty `since`) and `d3-waiting-turn-end-transcript-fallback-not-signal-only`, all `design` at MEDIUM.
Non-gating: the precedence contradiction with the output-files-first statements, the per-reader newest turn start, and the chunking measurement that belongs in `support-log.md`.
