---
round: 1
ledger:
  - key: friction-probe-runs-under-skip-git
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resume-friction-commit-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: anchored-path-has-content-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: bootstrap-commit-paths-comment-condition
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

First round, no prior ledger.
Four findings, all LOW or NIT: the probe runs under `FABRIC_SKIP_GIT`, the docs omit the resume commit, the probe lacks a table test, and one test comment misstates the condition.
