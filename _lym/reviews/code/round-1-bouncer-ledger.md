---
round: 1
ledger:
  - key: overview-stale-codex-no-skills-guards
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: prepare-gate-config-load-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: deny-output-and-matcher-not-pinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: prepare-python-publish-switches-unseparated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: skillload-branches-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: compareversions-not-total-order
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: skill-root-glob-metachar-hides-skills
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: classify-depends-on-cache-at-turn-end
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: codex-yaml-loaded-twice
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 (first ledger): F1 to F9 map to the keys above in order.
Gating: `prepare-gate-config-load-untested` (F2) and `deny-output-and-matcher-not-pinned` (F3).
F6 and F8 change how versions order and how the load line is matched, so their fixes are new text; the rest are local corrections the finding text fixes.
