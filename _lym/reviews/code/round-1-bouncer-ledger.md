---
round: 1
ledger:
  - key: loadenv-comment-omits-count-flag
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: testcacheoffflag-doc-overclaims-reach
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: withtestcacheoff-case-insensitive-key-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings, none gating.
F1 (`loadEnv` comment omits the leading `-count=1`), F2 (`TestCacheOffFlag` doc over-claims its reach) and F3 (no test row for the case-insensitive GOFLAGS key match) each have a local fix the finding text already states.
No earlier ledger exists.
