---
round: 2
ledger:
  - key: codex-weft-root-derived-in-engine
    rounds: [1]
    status: resolved
  - key: modelspec-template-comment-stale-aliases
    rounds: [1]
    status: resolved
  - key: codex-prepare-launch-shapes-untested
    rounds: [1]
    status: resolved
  - key: codex-records-root-not-symlink-resolved
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: configdirs-doc-weft-stat-unlogged
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's three keys are resolved: round 2 does not raise them, and its focus-question notes confirm the weft root is told geometry and the launch shapes are covered.
Round 2 mapping: F1 to `codex-records-root-not-symlink-resolved`, F2 to `configdirs-doc-weft-stat-unlogged`.
Both are local corrections whose result the finding text fixes.
