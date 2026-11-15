---
round: 1
ledger:
  - key: raw-dotnet-test-offered-despite-refuse-raw
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-test-line-misses-skipped-classes
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-test-notice-missing-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: codelocation-absent-code-dir-no-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 opened four findings.
F1 (`raw-dotnet-test-offered-despite-refuse-raw`) says the stencils and the no-test line's way forward name a raw `dotnet test` run that a hub's `refuse_raw` may deny.
F2 (`no-test-line-misses-skipped-classes`) says the line fires only on a whole-invocation zero and names no skipped class or reason.
F3 (`no-test-notice-missing-rows`) asks for rows covering a failing invocation and an invocation with no targets.
F4 (`codelocation-absent-code-dir-no-row`) asks for a row covering `CodeLocation` with an absent paired code directory.
