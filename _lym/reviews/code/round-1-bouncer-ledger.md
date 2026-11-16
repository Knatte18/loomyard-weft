---
round: 1
ledger:
  - key: run-field-key-literals
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: package-doc-profile-in-entry-cell
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
`run-field-key-literals` (F1) is the `pull_request` and `on_land` board.json key names written as repeated string literals in `runFieldLines`; the fix is two named constants referenced at every site.
`package-doc-profile-in-entry-cell` (F2) is a `doc.go` sentence that places the profile in the entry cell although it renders in the Recipe cell.
