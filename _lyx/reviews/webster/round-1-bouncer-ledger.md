---
round: 1
ledger:
  - key: comment-break-at-non-clause-comma
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: pushpairanchored-doc-old-side-names
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: package-doc-old-field-names
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: old-function-names-in-tests
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: fabricengine-budget-row-off-by-one
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: new-gate-refusals-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised six findings, all open.
F1 (`comment-break-at-non-clause-comma`) is the blocker: two new Go comments in `add.go` and `destroy.go` break at a comma that joins a compound object.
F2 is a stale doc on `PushPairAnchored` naming the sides warp or weft instead of code or records.
F3 and F4 are leftovers of the rename (package doc fields, old names in tests and a helper).
F5 is the test-budget row one above the package's top-level test count.
F6 is missing coverage for three new gate refusals.
