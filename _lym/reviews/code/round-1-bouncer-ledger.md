---
round: 1
ledger:
  - key: codex-weft-root-derived-in-engine
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: modelspec-template-comment-stale-aliases
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: codex-prepare-launch-shapes-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 mapping: F1 to `codex-weft-root-derived-in-engine`, F2 to `modelspec-template-comment-stale-aliases`, F3 to `codex-prepare-launch-shapes-untested`.
F1 changes who supplies the weft writable root, an interface change, so its fix needs a fresh review.
F2 and F3 are local corrections whose result the finding text fixes.
