---
round: 2
ledger:
  - key: dropped-handle-create-still-blocks-batch
    rounds: [1]
    status: resolved
  - key: decision-record-read-error-no-way-forward
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rename-not-done-offered-drop-route
    rounds: [1]
    status: resolved
  - key: pending-amendments-vanish-from-prompt
    rounds: [1]
    status: resolved
  - key: amendment-matcher-admits-unnamed-files
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: consolidation-strips-origin-webster
    rounds: [1]
    status: resolved
  - key: websterengine-doc-contradicts-loom-verb-texts
    rounds: [1]
    status: resolved
  - key: parent-fold-rule-loses-dropped-create
    rounds: [1]
    status: resolved
  - key: halt-reason-delivery-untested
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: selective-dropping-untested
    rounds: [1]
    status: resolved
  - key: verb-level-drop-path-untested
    rounds: [1]
    status: resolved
  - key: none-rendering-check-removed
    rounds: [1]
    status: resolved
  - key: webster-exemption-unbounded
    rounds: [1]
    status: resolved
  - key: parse-added-decisions-needs-keep-marker
    rounds: [1]
    status: resolved
  - key: decision-add-help-dropped-live-rules
    rounds: [1]
    status: resolved
  - key: validate-review-restates-plan-run-subdir
    rounds: [1]
    status: resolved
---

Round 1 raised 16 findings, two BLOCKING; round 2 resolved 13 of them.
Three carry into round 2 at LOW, all design class with local fixes.
The read-error way forward is fixed, but the wrapped error still documents an unreachable heading branch and names a webster verb on weaver paths.
The amendment exemption still admits the plan overview.
No test pins that only the plan review row gets the amendment reader.
