---
round: 1
ledger:
  - key: drift-gate-passes-on-fault
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pattern-departure-unknown-entry
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: empty-key-group-identity
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: records-mismatch-names-refused-verb
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: expect-seal-without-spawn-unenforced
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: test-keeps-removed-verb-name
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, one gating.

- `drift-gate-passes-on-fault` (F1): `NewDriftGate` in `internal/driftengine/check.go` withholds its failure when a records or seal fault is present.
- `pattern-departure-unknown-entry` (F2): two commits cite a Pattern-Departure entry that does not exist.
- `empty-key-group-identity` (F3): a key pattern matching an empty key group yields an uncoverable identity.
- `records-mismatch-names-refused-verb` (F4): a records mismatch at Drift-Expect or Drift-Baseline names `--restore`, which amend refuses there.
- `expect-seal-without-spawn-unenforced` (F5): a closed session mark cannot tell a normal end from a never-opened session.
- `test-keeps-removed-verb-name` (F6): `TestDeferWaive` keeps the removed waive verb's name.
