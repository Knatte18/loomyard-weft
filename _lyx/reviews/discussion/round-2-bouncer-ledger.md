---
round: 2
ledger:
  - key: d1-fixed-cache-ttl-premise
    rounds: [1]
    status: resolved
  - key: d2-bound-overstates-probe-reach
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d11-proof-does-not-isolate-hop
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: failed-ping-wrong-way-forward
    rounds: [1]
    status: resolved
  - key: ping-vs-reviewer-liveness-hold
    rounds: [1]
    status: resolved
  - key: d5-slug-from-other-task-worktree
    rounds: [1]
    status: resolved
  - key: d10-in-flight-claim-wrong-runs
    rounds: [1]
    status: resolved
  - key: uncapped-ping-count-off-by-one
    rounds: [1]
    status: resolved
  - key: scope-only-judging-decided-and-unconfirmed
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: ledger-interval-location-contradicts-d1
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d1-interval-precedent-and-never-pinged
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eight findings; D1's cache-lifetime premise was the blocker and its fix held, as did the fixes for the four smaller round-1 keys.
Round 2 shows two round-1 keys reopened by the fixes themselves.
D2's Bound now claims the request is marked expired on a failing tree, but `evaluateGate` stops at the first failing entry and never runs `parent-review`'s `Final`, so this is BLOCKING.
D11 still overclaims what `delivery.json` records: no caller identity and no proof of a send.
Three consistency findings are new: the stale scope-only "unconfirmed" text, the support-log ledger line on where the interval lives, and D1's precedent clause with its "never pinged" wording.
