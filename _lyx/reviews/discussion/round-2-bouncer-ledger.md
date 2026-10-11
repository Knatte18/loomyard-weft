---
round: 2
ledger:
  - key: driverless-seed-run-still-refused
    rounds: [1]
    status: resolved
  - key: abandoned-turn-gap-not-caught
    rounds: [1]
    status: resolved
  - key: compaction-becomes-idle
    rounds: [1]
    status: resolved
  - key: placeholder-parent-branch-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-no-timestamp-transcript-undecided
    rounds: [1]
    status: resolved
  - key: rejected-alternative-in-decisions
    rounds: [1]
    status: resolved
---

Round 1 raised six findings; round 2 raised one.
Five round 1 keys did not recur and the round 2 focus verdicts found the matching decisions consistent, so they are resolved.
The driverless seed order, D1's turn-start rule and compaction handling, and D2's empty-transcript case are settled.

The parent-branch key reopens as F1.
D3's Bound claims a wrong-parent seed strands the pair because no verb removes a seed.
The seed is an uncommitted file a plain `rm` clears, and the refusal on that path names a looping way forward.
The open question is which way forward D3 states, and whether `step`'s seed-stage refusal names it.
