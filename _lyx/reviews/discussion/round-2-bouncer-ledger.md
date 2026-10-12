---
round: 2
ledger:
  - key: d1-nonexistent-halted-state
    rounds: [1]
    status: resolved
  - key: d2-held-set-omits-asking
    rounds: [1]
    status: resolved
  - key: d3-no-cap-bound-false-premise
    rounds: [1]
    status: resolved
  - key: d4-global-flag-last-argv-only
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: prerun-rerun-risk-miscounted
    rounds: [1]
    status: resolved
  - key: d4-board-wording-departure-narrative
    rounds: [1]
    status: resolved
  - key: d2-acceptance-garbled-phrase
    rounds: [1]
    status: resolved
  - key: q2-stale-covered-run-list
    rounds: [1]
    status: resolved
  - key: d2-held-marker-exclusion-unexplained
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d3-bound-text-author-misstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised eight findings; round 2 raised three, and none of the round-1 subjects recurs except D4's global-flag forwarding.
That one is reopened: the fix handled `-v` and `--verbose` as exact words, but the root's count flag also takes `-vv` and `--verbose=<n>`, and `--json=<value>` escapes the help interception the same way.
It is the only gating item, at MEDIUM.
Two LOW findings are new: D2's Rationale claims to match shuttle's waiting reading while leaving out the held-turn wait marker without a reason, and D3's Bound line misstates who authors the pasted text and how name repair is gated.
