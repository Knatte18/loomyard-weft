---
round: 1
ledger:
  - key: detached-stop-cwd-not-anchor
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: beamdown-in-session-doc-claim-false
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stop-all-command-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: runhubstop-failure-paths-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: beamdown-skips-endorch-after-dobby-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-beam-downed-as-task-pair
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beamdown-endorch-error-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pause-lifted-true-after-operator-resume
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: hibernatecli-tree-line-misses-pausehub
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings, one BLOCKING.
The BLOCKING finding is the detached stop's working directory: it must be the prime's anchor, not its worktree root.
The MEDIUM findings are a false doc claim about a caller inside the orch's session and two test gaps in the stop --all paths.
The rest are an orch-process-end skip after a dobby error, worktree classification of an unreadable beam, one missing test row, a `pause_lifted` misreport and a docs tree line.
Every fix is local and its result is fixed by the finding text.
