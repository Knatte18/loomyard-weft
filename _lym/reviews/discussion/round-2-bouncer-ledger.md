---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's eleven findings are all resolved by round 2's review; its F7 answer notes the fix-here count it added.
Round 2 raised one finding, F12: that count is cleared "when an episode of PR-Review's segment begins", but the row declares no segment and `PRReview.Call` sees no episode boundary.
Either reading breaks D4's bound: carried across a send-back, it re-sends the run until the reroute cap; cleared on every re-step, it loses the per-episode cap.
The review's fix ties the reset to the send-back itself, which changes a decision and so needs a fresh review.
