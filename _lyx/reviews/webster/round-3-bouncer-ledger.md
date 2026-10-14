---
round: 3
ledger:
  - key: gates-header-closure-function-pairing
    rounds: [1]
    status: resolved
  - key: refusal-row-splits-circling-group
    rounds: [1]
    status: resolved
  - key: smoke-self-heal-live-driver-index-lock
    rounds: [2]
    status: resolved
  - key: smoke-self-heal-driver-git-child-index-lock
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two non-gating consistency findings, both resolved since round 2.
Round 2 raised the BLOCKING `smoke-self-heal-live-driver-index-lock`; the fix kills the live drivers at the top of the self-heal subtest, and round 3 does not raise it again, so it is resolved.

Round 3 raises one LOW `design` finding, `smoke-self-heal-driver-git-child-index-lock` (F1).
`killDriversAndClearLog` waits for the run and driver locks but not for a git child the killed driver was running, which can still hold the records `index.lock`.
The fix is local and fixed by the finding text: after `waitRunLockFree`, poll until the records worktree's `index.lock` is gone, with a short deadline.
