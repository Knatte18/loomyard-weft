---
round: 3
ledger:
  - key: tighter-read-rules-fixtures-untargeted
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: usage-entry-dropped-on-entry-time-attach
    rounds: [1]
    status: resolved
  - key: clean-converged-row-duplicates-verdicts-test
    rounds: [1]
    status: resolved
  - key: construction-refusal-outside-failure-table
    rounds: [1]
    status: resolved
---

Round 1 raised four keys; three were resolved by round 2.
`tighter-read-rules-fixtures-untargeted` recurs every round: round 1 missed the review-side read and usage record fixtures, round 2 the ledger-side read under CONVERGED, round 3 the CIRCLING replay rows of `TestBouncer_CirclingGuard` that drop a previous key.
Each round narrows the set but the enumeration method still misses a settle path, so the key is open at BLOCKING.
