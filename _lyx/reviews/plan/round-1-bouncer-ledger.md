---
round: 1
ledger:
  - key: verify-pause-hold-test-waits-production-durations
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: verify-entry-under-pause-skips-dirty-recheck
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: undeclared-pattern-cuts-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: twice-refused-resume-line-strands-session
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: sendconditioned-replays-read-coverage
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: start-color-line-refused-under-late-pause
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: unreadable-record-writes-paused-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: failed-group-signal-branch-unreachable
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: creel-doc-seam-sentence-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings, none carried from an earlier round.
The gating ones are two BLOCKING design findings on card 3 (`verifytree` pause hold): no time seam for its tests, and no dirty recheck after the wait.
Two MEDIUM decision findings follow: card 8's and card 9's PATTERN departures need a recorded decision, and card 14's second refusal must claim the session.
The remaining five are LOW or NIT local corrections.
