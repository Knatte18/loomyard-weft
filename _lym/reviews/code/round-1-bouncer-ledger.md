---
round: 1
ledger:
  - key: orch-engine-hardcoded-literal
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: stencilkit-duplicate-edit-directive-block
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: tidy-runner-build-error-masked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: producer-edit-directive-copy-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: loom-driver-render-unexported
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: banners-still-say-shared-edit-directive
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orch-remote-control-codex-default-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: fixer-spec-prompt-engine-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: weaver-per-rung-engine-first-rung-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: contained-tests-lack-keep-marks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings: F1 to F10 in review order, keyed above.
F1 and F2 are BLOCKING on `PATTERN-code-conventions`; F3 and F4 are gating MEDIUM design findings.
F1 and F3 change an approach or an interface, so their fixes need a fresh review; the rest are local corrections.
