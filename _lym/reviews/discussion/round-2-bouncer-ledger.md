---
round: 2
ledger:
  - key: d4-memory-quotient-no-floor
    rounds: [1]
    status: resolved
  - key: d3-prebuilt-lym-reaches-no-test
    rounds: [1]
    status: resolved
  - key: d4-two-tier-concurrency-open
    rounds: [1]
    status: resolved
  - key: d6-precheck-order-and-dropped-bound
    rounds: [1]
    status: resolved
  - key: d1-bound-misstates-skipped-features
    rounds: [1]
    status: resolved
  - key: d1-package-set-misses-deleted-tests
    rounds: [1]
    status: resolved
  - key: d9-retargets-one-of-three-refusals
    rounds: [1]
    status: resolved
  - key: d6-nested-package-files-counted
    rounds: [1]
    status: resolved
  - key: d7-contradicts-d6-on-maps-written
    rounds: [1]
    status: resolved
  - key: gate-slot-background-does-not-admit-map-run
    rounds: [1]
    status: resolved
  - key: d5-rerun-list-leaves-no-trace
    rounds: [1]
    status: resolved
  - key: d6-failed-listing-written-unchecked
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: map-run-time-bounds-unstated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-d8-whole-replace-vs-d7-keeps-entries
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-memory-bound-needs-tree-cap-host
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-gitignored-files-escape-checks
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-green-skip-criterion-ignores-earlier-record
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-slot-ask-leaves-all-unranked
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's eleven keys do not recur in round 2, so all are resolved; the two BLOCKING defects (D4's memory floor, D3's prebuilt `lym`) are closed.
Round 2 raises seven new keys, none BLOCKING.
Two are gating MEDIUM design findings on first-raised keys: a failed linked-package listing written with no check (F1), and the unstated time bounds of a map run (F2).
Go carries them into `## Open risks`.
The five others are LOW or NIT, four of them consistency corrections whose text the findings already fix, and one LOW design finding on gitignored files (F5) whose fix is a single Bound sentence.
