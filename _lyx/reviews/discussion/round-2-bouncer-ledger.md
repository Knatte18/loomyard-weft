---
round: 2
ledger:
  - key: d1-handon-signal-matches-d2-faults
    rounds: [1]
    status: resolved
  - key: d1-settling-fork-unbounded
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: d3-peer-forks-unlinked-to-d1-d2
    rounds: [1]
    status: resolved
  - key: d7-issue-673-move-misses-post-plan-landing
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-bound-overstates-no-override
    rounds: [1]
    status: resolved
  - key: d2-rationale-deliberation-narrative
    rounds: [1]
    status: resolved
  - key: d1-brief-step1-invites-fork-design-calls
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings; round 2 raised three.
Four round 1 keys no longer apply: the D1 hand-on overlap with D2 (round 2 shows the sets are disjoint), the D3 peer-fork linkage, the D6 bound wording and the D2 rationale narrative.

Two round 1 keys persist.
`d1-settling-fork-unbounded` narrowed to one hole: "recorded operator decision" does not exclude arbiter, parent and auto-mode entries.
`d7-issue-673-move-misses-post-plan-landing` reopened as a design gap: D7 hands the dropped #673 entry to a Merge-In resolution that never runs on a clean merge, so no actor owns it.

One new key, `d1-brief-step1-invites-fork-design-calls`, is a local wording conflict between the escalation brief's step 1 and D1's settle rule.
