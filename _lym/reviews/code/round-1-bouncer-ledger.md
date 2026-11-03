---
round: 1
ledger:
  - key: last-envelope-selection-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: open-worktree-error-always-lasting
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rune-cut-not-exercised
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, none gating.
F1 asks for a `failures` row pinning last-envelope selection (key `last-envelope-selection-unpinned`).
F2 narrows the lasting mark in `runCaptureChild` to the two cwd refusals (key `open-worktree-error-always-lasting`).
F3 asks for a multibyte row pinning the rune-based cut (key `rune-cut-not-exercised`).
