---
round: 3
ledger:
  - key: d3-trailer-way-forward-reuses-uncaused-ruling
    rounds: [1]
    status: resolved
  - key: d4-sendback-file-archived-by-new-generation
    rounds: [1]
    status: resolved
  - key: d4-sendback-into-disabled-perch
    rounds: [1]
    status: resolved
  - key: d4-full-generation-review-input-not-existing
    rounds: [1]
    status: resolved
  - key: d4-parent-review-asked-flag-not-cleared
    rounds: [1]
    status: resolved
  - key: d4-reentry-session-resumes-recorded-id
    rounds: [1]
    status: resolved
  - key: d4-verb-skip-or-shaping-unbounded
    rounds: [1]
    status: resolved
  - key: d2-trailer-cause-unverified-no-bound-named
    rounds: [1]
    status: resolved
  - key: d4-dirty-tree-refusal-only-at-verb
    rounds: [1]
    status: resolved
  - key: d2-go-appended-trailer-no-criterion
    rounds: [1]
    status: resolved
  - key: auto-mode-assumption-duplicates-d4
    rounds: [1]
    status: resolved
  - key: d4-fix-here-count-no-defined-reset-point
    rounds: [2]
    status: resolved
  - key: d4-clear-before-reroute-persisted
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's eleven findings were resolved by round 2.
Round 2's F12, the undefined reset point of the fix-here count, is resolved in round 3: D4 now has the row clear the count with the mark on a clean-tree reroute.
Round 3 raises F13: that clear runs inside the row's `Call`, before shed persists the reroute, so a crash in between drops the send-back and resets the count.
The review's fix moves the clear to the start of the row's next `Call`, gated on the newest shed history entry being the reroute entry, which changes a decision and needs a fresh review.
