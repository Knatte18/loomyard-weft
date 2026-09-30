---
round: 1
ledger:
  - key: probe-error-vs-half-broken-pair-fallback
    rounds: [1]
    status: resolved
  - key: mergeinprogress-rejection-rationale
    rounds: [1]
    status: resolved
  - key: batten-retry-verified-not-assumed
    rounds: [1]
    status: resolved
  - key: driver-not-parked-precedence
    rounds: [1]
    status: resolved
  - key: conflicted-path-form
    rounds: [1]
    status: resolved
  - key: goal-sentence-wording
    rounds: [1]
    status: resolved
---

In round 1 the reviewer raised six findings against the discussion pair.
The most serious (MEDIUM) was a contradiction: an unopenable pair could have been reported as "not mid-merge", which would have let `start` spawn a driver on an unchecked tree.
The five smaller findings were an inaccurate reason for rejecting `Fabric.MergeInProgress`, batten's non-retryable handling left as an assumption, unstated precedence against `driver_not_parked`, an unspecified form for the conflicted paths, and ambiguous wording in the Goal.
The current artifacts reflect all six, so every entry is resolved and none is open for round 2.
