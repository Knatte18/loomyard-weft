---
round: 1
ledger:
  - key: glyph-parse-outside-planparser
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: undeclared-pattern-edits
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: false-language-mismatch-unresolved-member
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: csharp-constructor-rename-accepted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: plandag-csharp-test-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-language-recognized-doc
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings.
F1 (BLOCKING) is the C# re-sign path in `internal/planglyph/resign.go` parsing with `glyph.CSharp` outside `internal/planparser`; the fix is a `planparser` helper.
F2 reverts two PATTERN entries edited without a declaration.
F3 to F6 are local corrections.
