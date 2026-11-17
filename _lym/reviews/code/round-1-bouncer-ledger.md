---
round: 1
ledger:
  - key: scope-never-starts-reads-as-failing-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: memory-scope-probe-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rlimit-data-replaces-rlimit-as-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: testmap-capped-runs-lack-parallel-bound
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: runalone-oom-branch-untested-on-scope-host
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: report-step-and-testmap-capped-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: spawn-exemption-reason-misses-probe
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: template-comment-misstates-capped-handling
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: windows-job-joined-after-start
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: testtiming-bounds-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists and nothing recurs.
Ten findings are open: F1 to F10 in the order of the ledger above.
Four are gating (F1, F2, F5, F6).
F3 is a recorded-decision gap that the run's parent settles with `lym loom decision add`, and F4 is a scope question the parent may also settle by recording instead of a code change.
