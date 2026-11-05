---
round: 1
ledger:
  - key: stray-delete-bypasses-destruction-gate
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: teardown-request-clearstray-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pairparts-straypath-untested-through-inspectpair
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: beamland-member-listing-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: remove-stray-error-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cancel-remove-stray-delete-unreported
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review.

- F1 is the one design question: `RemoveStrayTaskPath` runs outside the `PATTERN-fabric-git` destruction chokepoint's single exception, so the code must route through the gate or the PATTERN text must be amended with a `Pattern-Departure:` trailer.
- F2, F3 and F4 are missing test rows for `teardownRequest`, `PairParts.StrayPath` and the beam-land member listing.
- F5 is missing rows for error-path branches of `Remove` and `RemoveStrayTaskPath`.
- F6 asks the cancel report to name a deleted stray directory.
