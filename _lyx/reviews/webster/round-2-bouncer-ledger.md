---
round: 2
ledger:
  - key: gates-header-closure-function-pairing
    rounds: [1]
    status: resolved
  - key: refusal-row-splits-circling-group
    rounds: [1]
    status: resolved
  - key: smoke-self-heal-live-driver-index-lock
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised two non-gating consistency findings: a `gates.go` header comment that lists one function fewer than closures, and a `validate-discussion` row placed between two circling rows in the refusal spec.
Round 2 does not raise either again, so both are resolved.

Round 2 raised one new finding, `smoke-self-heal-live-driver-index-lock` (F1, BLOCKING, `design`).
The subtest "origin record self-heals after a crash between write and commit" runs `git rm` in the records worktree while the driver left running by the preceding subtest can hold the weft `index.lock`.
The fix is local and fixed by the finding text: call `killDriversAndClearLog(t)` as the subtest's first statement.
