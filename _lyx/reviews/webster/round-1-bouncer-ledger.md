---
round: 1
ledger:
  - key: gates-header-closure-function-pairing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refusal-row-splits-circling-group
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two non-gating consistency findings.
F1 (`gates-header-closure-function-pairing`) is a header comment in `internal/loomshed/gates.go` that lists one function fewer than closures.
F2 (`refusal-row-splits-circling-group`) is a `validate-discussion` row in `contracts/specs/refusal-spec.md` placed between two circling rows.
Both are local corrections whose result the finding text fixes.
