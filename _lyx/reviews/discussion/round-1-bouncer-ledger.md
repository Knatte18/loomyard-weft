---
round: 1
ledger:
  - key: selfexec-leaf-set-outside-pattern
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: replace-selection-moves-viewer-off-shell
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: hidden-window-insert-target-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: no-shell-fallback-order-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: migration-step-failure-bound-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-sidecar-consequence-only-in-support-log
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: scope-out-carries-history
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: migration-criterion-says-last
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, all open.
The gating ones are the `internal/selfexec` leaf set kept outside `PATTERN-leaf-packages`, the target-less hidden-window insert, the unordered no-shell fallback and the unbounded migration step.
The viewer-off-shell conflict between D1 and D5 is a consistency finding, not gating.
