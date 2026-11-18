---
round: 1
ledger:
  - key: python-hook-bare-lym-path
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: standalone-test-misses-real-branch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: loadconfig-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: hub-test-differs-from-launch-mode
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: codex-overview-paragraph-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pythonallowedcheck-broken-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-omits-standalone-fallback
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-row-unenterable-project-dir
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: dead-pythondeny-matrix-column
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings.
F1 (python-hook-bare-lym-path) is BLOCKING: both engines' python hook commands run `lym` from PATH instead of resolving it through `selfexec.LymBinary`.
F2 and F3 are MEDIUM test gaps in the verb's standalone and config-failure paths.
The rest are LOW or NIT: hub-mode test mismatch, a stale Codex doc paragraph, a broken doc sentence, an incomplete change description, a missing project-directory row and a dead matrix column.
