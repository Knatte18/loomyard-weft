---
round: 4
ledger:
  - key: tighter-read-rules-fixtures-untargeted
    rounds: [1, 2, 3]
    status: resolved
  - key: usage-entry-dropped-on-entry-time-attach
    rounds: [1]
    status: resolved
  - key: clean-converged-row-duplicates-verdicts-test
    rounds: [1]
    status: resolved
  - key: construction-refusal-outside-failure-table
    rounds: [1]
    status: resolved
  - key: replay-guard-exemption-keys-on-escalation-only
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: lowered-beta-drops-blocking-carry-over-test
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 to 3 raised `tighter-read-rules-fixtures-untargeted` each round, each time missing one settle path. Round 4's enumeration of every keyed fixture found no further gap, so it is resolved.
Round 4 raised two new keys: the replay guard's exemption ignores a recorded decision with no escalation record (MEDIUM, design), and lowering beta to MEDIUM leaves no test of BLOCKING carry-over (LOW, consistency).
