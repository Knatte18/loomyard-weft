---
round: 1
ledger:
  - key: end-live-relays-stops-at-first-failure
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-app-id-passes-app-match
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: live-hub-geometries-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ack-comment-names-unstated-deadline
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, all local corrections whose fix text is fixed by the review.
F1 maps to the relay-ending loop returning at the first failed stop, F2 to the empty app id passing the match,
F3 to the missing test for the live-hub seam, and F4 to the Ack comment's three-second claim.
