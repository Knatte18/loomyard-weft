---
round: 1
ledger:
  - key: merriam-base-reads-worktree-root
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: wiring-anchored-exclude-lines-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: heddle-help-and-doc-say-root
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: heddle-render-show-verbs-no-anchored-test
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
Four findings were raised: F1 (Merriam base at worktree root), F2 (wiring's anchored exclude lines untested), F3 (help text and package doc still say root) and F4 (render and show verbs lack an anchored test).
None is gating, and each fix is a local correction its finding text fully specifies.
