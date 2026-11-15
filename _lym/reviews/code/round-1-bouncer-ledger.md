---
round: 1
ledger:
  - key: impactset-importers-binarybuilders-no-reader
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: ahead-coverage-build-not-cancelled
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ahead-build-overlap-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: doc-side-by-side-workdir-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four LOW findings and no BLOCKING or MEDIUM ones.
F1 is leftover coverage-scope support in `internal/impactset`.
F2 is an unused ahead build waited for instead of cancelled.
F3 is the missing test for the ahead-build overlap.
F4 is a wrong reason in the `internal/gatecli/doc.go` side-by-side justification.
There was no earlier ledger, so nothing carries forward.
