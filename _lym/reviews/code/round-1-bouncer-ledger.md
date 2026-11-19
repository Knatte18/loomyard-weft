---
round: 1
ledger:
  - key: embedded-bytes-fallback-against-stencil-ownership
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: skew-tested-only-through-unexported-helper
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: edited-copy-from-newer-writer-bypasses-detection
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: writer-ordering-disagrees-with-refresh-ordering
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: registry-doc-says-read-needs-no-registry
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
The central one is F1: the skew fallback renders embedded registry bytes, which `PATTERN-stencil-ownership` forbids outside the orch role.
Its fix replaces the fallback with an early refusal, and F2, F3 and F5 depend on that new shape.
F4 adds a registered ordering so the read path agrees with the refresh's ancestry-first ordering.
