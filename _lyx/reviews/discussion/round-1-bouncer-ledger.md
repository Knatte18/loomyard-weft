---
round: 1
ledger:
  - key: relaunch-wording-no-third-file-carrier
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: self-check-passes-skipped-relevant-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-never-search-further-subject
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: path-rule-bound-overclaims
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: duplicate-head-start-descriptions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
F1 is the only gating one: D1's relaunch wording names three outputs, but the shared twill relaunch stencil carries no file names, so the decision needs an explicit mechanism or a retraction.
F4 changes the path-rule decision, so it is `design-wrong`.
F2, F3 and F5 are local wording corrections the review text already fixes.
