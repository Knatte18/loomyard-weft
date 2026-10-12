---
round: 1
ledger:
  - key: hub-pause-caller-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: parent-lock-exception-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: covered-lock-holder-freezes-others
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: orch-interrupt-skips-grace-and-report
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: orch-resume-line-ownership
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: loom-lock-queue-no-way-out
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: post-kill-stop-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: continue-line-no-prewrite-guard
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: goal-overstates-boundary-pause-removal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d4-rearg-rejected-interrupt-alternative
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: question-ledger-stale-open-risks-pointer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eleven findings, none carried from earlier rounds.
The gating cluster is unbounded new capability: a hub-wide pause any agent can issue, a parent-lock exception that lets a covered run land, and a covered lock holder that freezes other runs.
The remaining design findings leave the orch interrupt, the loom lock-queue way forward and the post-kill stop without a stated bound.
The consistency findings are text fixes with no design weight.
