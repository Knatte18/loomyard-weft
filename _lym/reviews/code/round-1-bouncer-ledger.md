---
round: 1
ledger:
  - key: shipped-slug-collision-stuck
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bad-override-file-fails-all-reads
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: show-passages-override-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: override-file-not-claimed-by-replayer
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shipped-layer-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: batching-sentence-not-pinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: claudeengine-doc-stale-passages-source
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: configcli-unrelated-help-edit
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so there is no prior ledger.
Three MEDIUM design findings concern heddleengine: the shipped-slug refusal ordering (F1), the override load failure path (F2) and missing Show and Passages override tests (F3).
The LOW findings are replayer claims (F4), untested failure and default paths (F5) and an unpinned batching sentence (F6).
The NITs are a stale claudeengine doc sentence (F7) and an unrelated configcli help edit (F8).
