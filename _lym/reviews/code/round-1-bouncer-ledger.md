---
round: 1
ledger:
  - key: fixer-no-commit-leaves-judged-empty
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rebase-after-parent-merge-noop-and-mismatch-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fix-review-lock-free-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: closed-watch-channel-collapses-backoff
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: front-priority-lost-across-resume
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pause-holds-fixer-gate-calls-unstated
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: requeue-after-merge-in-required-retry-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: judged-covered-on-within-caps-route-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: front-ticket-ordering-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: watch-open-failure-fallback-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-queue-head-wording-on-store-covered
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eleven findings, none carried from an earlier round.
Three are gating: a correctness gap in `judgeFixerChange` (F1) and two test gaps (F2, F3) around the rebase after a parent merge and the lock-free fix review.
The remaining LOW findings and the NIT cover the closed-watch backoff, front priority across a resume, an unstated pause effect, and further test and wording gaps.
Every fix is a local correction its finding text already fixes.
