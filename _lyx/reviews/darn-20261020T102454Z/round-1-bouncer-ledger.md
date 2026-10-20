---
round: 1
ledger:
  - key: impactset-command-runs-unit-tier-twice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: prebuilt-prune-removes-refreshed-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prebuilt-republish-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: verifygate-no-package-pass-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cross-cutting-rules-only-in-package-docs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: claude-md-push-rule-line-breaks
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, F1 to F6, in the order of the keys above.
F1 was relabeled from `scope` to `design`: it is the one MEDIUM finding and the reason for `CONTINUE`.
All six are local corrections whose result the finding text fixes.
