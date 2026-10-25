---
round: 2
ledger:
  - key: decision-halt-reaches-go-driver-tackler
    rounds: [1]
    status: resolved
  - key: decision-accept-drops-blocking-findings-from-carry-over
    rounds: [1]
    status: resolved
  - key: card16-misses-old-gate-key-fixture-literals
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: settled-decision-accept-replay-re-judged
    rounds: [1]
    status: resolved
  - key: held-dropped-key-refusal-names-continue
    rounds: [1]
    status: resolved
  - key: decision-findings-unrouted-on-handled-false-paths
    rounds: [1]
    status: resolved
  - key: darn-writer-no-parent-never-rechecks-note
    rounds: [1]
    status: resolved
  - key: pattern-cwd-resolution-keeps-deleted-package
    rounds: [1]
    status: resolved
  - key: card3-redundant-gate-list-test
    rounds: [1]
    status: resolved
  - key: docs-overview-keeps-retired-names
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card12-escalation-states-no-bound
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card7-asserts-lens-prose
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1's BLOCKING halt-routing finding and the rest of its keys are not re-raised in round 2, except the card 16 fixture gap, which remains at LOW with two more fixtures.
Round 2 adds a MEDIUM on two `docs/overview.md` lines that name retired names, and two LOW findings on card 12's missing bound and card 7's lens-prose assertions.
All round 2 fixes are local corrections whose text the findings fix.
