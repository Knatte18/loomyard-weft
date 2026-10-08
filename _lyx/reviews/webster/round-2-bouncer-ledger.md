---
round: 2
ledger:
  - key: serial-tests-no-named-reason
    rounds: [1]
    status: resolved
  - key: unsent-reprompt-cap-not-in-plan
    rounds: [1]
    status: resolved
  - key: send-go-header-narrower-than-file
    rounds: [1]
    status: resolved
  - key: leftover-strand-warn-on-every-reclaim
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: doc-restates-perishable-constants
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1's three findings are resolved: the two send tables run in parallel, the unsent re-prompt retry is bounded by the run deadline alone, and `send.go`'s header covers the whole file.
Round 2 raised two new keys, neither BLOCKING nor MEDIUM.
F1 (LOW, design) is `stopLeftoverStrand` logging its Warn for a strand that is not live, and doubling shuttle's own Warn for a live one.
F2 (NIT, consistency) is `doc.go` restating the values of `turnStartIdleOverride` and `confirmBackoffCap`.
