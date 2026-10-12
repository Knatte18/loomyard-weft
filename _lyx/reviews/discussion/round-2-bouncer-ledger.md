---
round: 2
ledger:
  - key: d1-interpreter-wording-fails-no-denied-recovery-scan
    rounds: [1]
    status: resolved
  - key: d6-latch-never-clears-after-first-notice
    rounds: [1]
    status: resolved
  - key: d6-restart-rebuild-repeats-sent-notice
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d2-sed-word-extent-and-incomplete-bound
    rounds: [1]
    status: resolved
  - key: d7-closed-list-covers-only-bare-in-place-spellings
    rounds: [1]
    status: resolved
  - key: d3-bound-omits-positional-parameters
    rounds: [1]
    status: resolved
  - key: goal-claims-more-steered-forms-than-d3-covers
    rounds: [1]
    status: resolved
  - key: d1-exclude-entry-has-no-stated-reach
    rounds: [1]
    status: resolved
  - key: d2-abbreviated-in-place-spellings-not-denied
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-latch-outlives-quickly-answered-dialog
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised eight findings; round 2 raised three, and the review reports none of round 1's other keys again.
The BLOCKING D1 wording finding and the round 1 D6 latch, D2 extent, D7, D3, Goal and D1 exclude findings are resolved.
The D6 restart rebuild reopens at LOW: an ask landing after the notice is sent repeats the notice on reattach.
New this round are the abbreviated `--in-place` gap in D2 (MEDIUM, design, carried to Open risks) and the D6 latch that outlives a quickly answered dialog (LOW, design).
