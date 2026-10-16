---
round: 2
ledger:
  - key: free-lock-vs-exiting-daemon
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d5-path-prepend-unbounded
    rounds: [1]
    status: resolved
  - key: d8-creel-check-no-strand
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: goal-680-broader-than-scope
    rounds: [1]
    status: resolved
  - key: bounds-without-criteria
    rounds: [1]
    status: resolved
  - key: d8-creel-closing-sentence-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's five keys: three were not raised again and are resolved.
`d8-creel-check-no-strand` reopened in round 2 as F1: D8 now says every creel strand starts a fresh shell, which the fake idle pane scripts contradict; the review spells out the replacement wording.
`free-lock-vs-exiting-daemon` reopened as F2: the Goal and D3's Rationale promise no watchdog process at `down`'s return, while D3, D3a and the first Acceptance criterion deliver only a free lock.
F3 is new: D8's "no other suite line changes" leaves the creel suite's closing pre-condition sentence stale.
