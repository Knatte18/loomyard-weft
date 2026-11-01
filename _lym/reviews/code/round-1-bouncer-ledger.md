---
round: 1
ledger:
  - key: outstanding-forks-drops-unresolved-launch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: await-forks-verb-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: runner-awaitforks-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: await-forks-no-exit-for-never-ready
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: turn-end-fork-launch-not-pinned-at-event
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: findrun-error-misreported
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, all class `design`.
Three are MEDIUM and gate: a fork launch without a recorded tool result is not counted as outstanding (F1), and neither the `await-forks` verb (F2) nor `Runner.AwaitForks` (F3) has a test.
F4 (unbounded re-runs in the stencils), F5 (turn-end reading unpinned at the event level) and F6 (misworded FindRun error) are LOW or NIT.
