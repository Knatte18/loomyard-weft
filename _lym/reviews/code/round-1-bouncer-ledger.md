---
round: 1
ledger:
  - key: scope-start-script-repeats-cgroup-path-literal
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: leftover-kill-test-real-time-bound
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: oom-kill-of-child-leaves-scope-running
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: oom-policy-continue-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: start-script-depends-on-path-cat
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: kill-scope-write-creates-and-truncates
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: vanished-scope-tolerance-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review; no earlier ledger exists.
Two BLOCKING consistency findings: a repeated cgroup path literal in the start script, and a 10 s real-time poll bound in the new test.
Two MEDIUM design findings: a child's OOM kill leaves the scope's other processes running under `OOMPolicy=continue`, and no test pins that policy.
Three LOW design findings: the PATH-dependent `cat`, `cgroup.kill` opened with create and truncate, and the unpinned vanished-scope tolerance.
