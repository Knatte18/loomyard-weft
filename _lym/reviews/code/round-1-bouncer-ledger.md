---
round: 1
ledger:
  - key: websterengine-loadstate-no-scratch-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: readorigin-no-weft-no-exclude-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: readjson-lock-error-bound-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: huborchs-staging-skip-and-silent-skip-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: verifyseedownership-redundant-mkdirall
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five non-gating findings: four test-coverage gaps against the acceptance criteria (F1 to F4) and one leftover `MkdirAll` in `VerifySeedOwnership` (F5).
Each fix is a local correction its finding text already fixes.
