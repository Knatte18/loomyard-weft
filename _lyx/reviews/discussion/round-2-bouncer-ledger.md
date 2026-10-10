---
round: 2
ledger:
  - key: advisor-start-failure-notice-before-chair-exists
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: send-notice-needs-idle-chair
    rounds: [1]
    status: resolved
  - key: done-advisor-strand-removed
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: advisor-to-parent-route-unbounded
    rounds: [1]
    status: resolved
  - key: still-live-undefined
    rounds: [1]
    status: resolved
  - key: provider-word-single-source-claims-false
    rounds: [1]
    status: resolved
  - key: added-after-discussion-duplicates-d6
    rounds: [1]
    status: resolved
  - key: decisions-relitigate-rejected-alternatives
    rounds: [1]
    status: resolved
  - key: send-notices-unserialized
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: chair-to-advisor-edge-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised eight findings; round 2 raised four.
Six round 1 keys were not raised again and are resolved.

Two round 1 keys remain open.
`done-advisor-strand-removed` returns as F1 (BLOCKING): the resume probe in D8 removes the strand of a Done advisor kept by `KeepPane` and reports it dead, contradicting D5.
`advisor-start-failure-notice-before-chair-exists` returns as F2 (MEDIUM): no Decision says when advisor joins begin or where a notice raised before the chair's handle exists goes.

Two keys are new in round 2.
`send-notices-unserialized` (F3) is D7's one goroutine per notice sending into the chair's pane without serialization.
`chair-to-advisor-edge-unbounded` (F4) is D6's parent directive rendered with the chair as parent, granting delegate authority to the chair with no stated bound.
