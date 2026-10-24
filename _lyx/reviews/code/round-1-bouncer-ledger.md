---
round: 1
ledger:
  - key: verbatim-string-escaped-quote-lexed-as-raw
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: record-identifier-parsed-as-type-decl
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gatecli-doc-omits-dotnet-whole-entry-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, none carried from earlier rounds.
F1 maps to `verbatim-string-escaped-quote-lexed-as-raw` (`internal/dotnetimpact/lex.go`), F2 to `record-identifier-parsed-as-type-decl` (`internal/dotnetimpact/source.go`), F3 to `gatecli-doc-omits-dotnet-whole-entry-only` (`internal/gatecli/doc.go`).
Each fix text states the exact local correction, so all three are mechanical.
