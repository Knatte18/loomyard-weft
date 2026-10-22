---
round: 2
ledger:
  - key: verify-pause-hold-test-waits-production-durations
    rounds: [1]
    status: resolved
  - key: verify-entry-under-pause-skips-dirty-recheck
    rounds: [1]
    status: resolved
  - key: undeclared-pattern-cuts-unrecorded
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: twice-refused-resume-line-strands-session
    rounds: [1]
    status: resolved
  - key: sendconditioned-replays-read-coverage
    rounds: [1]
    status: resolved
  - key: start-color-line-refused-under-late-pause
    rounds: [1]
    status: resolved
  - key: unreadable-record-writes-paused-marker
    rounds: [1]
    status: resolved
  - key: failed-group-signal-branch-unreachable
    rounds: [1]
    status: resolved
  - key: creel-doc-seam-sentence-stale
    rounds: [1]
    status: resolved
  - key: verify-hold-marker-not-removed-after-hold
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings; round 2 raised two.
Eight round-1 keys, including both BLOCKING design findings on card 3, were not raised again and are resolved.
`undeclared-pattern-cuts-unrecorded` recurs as F1 (decision, MEDIUM): the decision record lacks an `Added after Discussion` entry for card 8's cuts and card 9's shortened line, and the parent must add it.
F2 is new: card 3 leaves removal of the `paused` marker after a hold on `StatusDirty` or `StatusSkipped` returns unstated and unpinned.
