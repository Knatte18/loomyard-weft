---
round: 1
ledger:
  - key: decision-halt-reaches-go-driver-tackler
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: decision-accept-drops-blocking-findings-from-carry-over
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card16-misses-old-gate-key-fixture-literals
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: settled-decision-accept-replay-re-judged
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: held-dropped-key-refusal-names-continue
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: decision-findings-unrouted-on-handled-false-paths
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darn-writer-no-parent-never-rechecks-note
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-cwd-resolution-keeps-deleted-package
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-redundant-gate-list-test
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings, all open.
The BLOCKING one (F1) changes which predicate the loom spec and the Go driver use to hand back a decision halt, so its fix is design-wrong and needs a fresh review.
The rest are local corrections whose result the finding text fixes.
