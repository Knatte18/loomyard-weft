---
round: 1
ledger:
  - key: wired-landlocally-failed-head-read-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: parent-branch-read-after-status-commit
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: landing-commit-wording-overstates-rerun
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unrecorded-commit-assertion-vacuous
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 opened four findings.
The gating one is a coverage gap: the failed post-merge head read on a wired local landing.
The others are the order of the parent branch read against the status commit, with the ledger writer's signature;
doc wording that calls the post-merge head the run's own landing commit even on a re-run;
and a test assertion that passes under the wrong behaviour too.
