---
round: 1
ledger:
  - key: darn-recipe-red-first-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: overview-unjudged-check-holds-handoff-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: exit5-runner-crash-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-overstates-no-test-hold
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, none gating.
F1 (recipe comment) and F2 (overview) are stale prose about the exit-5 hold.
F3 is a missing test row for exit 5 with a runner crash.
F4 is a summary claim that the diff does not bear out.
