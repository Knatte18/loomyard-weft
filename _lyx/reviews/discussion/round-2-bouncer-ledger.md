---
round: 2
ledger:
  - key: d4-nil-cause-trigger-overbroad
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d2-background-omits-lone-percent-letthrough
    rounds: [1]
    status: resolved
  - key: d5-misreads-daemon-wakeups-scope
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-dismissal-gap-edges-unstated
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-operator-hold-overstated
    rounds: [1]
    status: resolved
  - key: d7-cap-governs-two-cases
    rounds: [1]
    status: resolved
  - key: d4-cause-present-tail-misdescribed
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised six findings; round 2 raised four.
Three of round 1's keys (D1, D2, D7) did not recur and are resolved.
D5 persists: the daemon-wakeups exemption reading lives only in the decision record and the floor argument misses the backoff rule.
D6 persists in a narrower form: a partial play that reached the pane starts no gap.
D4's trigger wording still conflicts with the retry exits (LOW), and a new D4 finding says delivery failures already carry a pane tail, contrary to the record's stated premise.
