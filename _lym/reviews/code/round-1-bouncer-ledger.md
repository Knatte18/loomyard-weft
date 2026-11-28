---
round: 1
ledger:
  - key: owed-only-check-not-rechecked-under-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: finalize-ahead-parent-upstream-fallbacks-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: parent-align-owed-ahead-upstream-tip-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: github-landing-merges-owed-tips-before-push-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: parentmergetip-doc-contradicts-owed-branch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opened every key; there is no earlier ledger.

The open work centres on the lock-free pass taking the local tip of a parent that is ahead only by owed landing tips:
the check that licenses that tip runs without the lock, the fallbacks to the upstream tip lack tests on both the Finalize and Parent-Align sides,
a landing that merges on GitHub now takes the owed tips before its push check, and the `parentMergeTip` doc contradicts the new branch.
