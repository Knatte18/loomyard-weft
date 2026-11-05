---
round: 1
ledger:
  - key: bolt-lock-path-not-refused
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: headroom-commit-dropped-unrelated-pattern-text
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: shedadapters-lock-move-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: refuse-tracked-tree-skips-after-earlier-lym-segment
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-push-lock-residue-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: moved-lock-behaviours-lack-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all open.
F1 is the gating one: the `_lym`-only refusal leaves a Bolt-side lock path accepted, so the new PATTERN entry's `(test)` claim is too wide.
F3 is the second gating finding, an untested lock move in `internal/shedadapters`.
F2 restores PATTERN text the headroom commit cut from untouched entries.
F4 to F6 are LOW cleanups in `internal/lock`, `internal/fabricengine` and test coverage.
