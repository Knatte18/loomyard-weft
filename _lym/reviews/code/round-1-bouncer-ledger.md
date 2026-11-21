---
round: 1
ledger:
  - key: dropped-handle-create-still-blocks-batch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: decision-record-read-error-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: rename-not-done-offered-drop-route
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pending-amendments-vanish-from-prompt
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: amendment-matcher-admits-unnamed-files
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: consolidation-strips-origin-webster
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: websterengine-doc-contradicts-loom-verb-texts
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: parent-fold-rule-loses-dropped-create
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: halt-reason-delivery-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: selective-dropping-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: verb-level-drop-path-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: none-rendering-check-removed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: webster-exemption-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: parse-added-decisions-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: decision-add-help-dropped-live-rules
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: validate-review-restates-plan-run-subdir
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
---

Round 1 raised 16 findings, none carried from an earlier round.
The two BLOCKING ones are F1 (a dropped handle-declared Create still fails the batch through `BindHandles`) and F2 (a decision-record read or heading error reaches the webster verbs with no way forward).
The design-class MEDIUM findings concern the way forward offered for a rename, the prompt dropping pending amendments, the amendment matcher's looseness, and three test gaps (F9, F10, F11).
The consistency findings concern stencil origin rules, a stale package doc, the parent's fold rule, a removed test check and help text.
