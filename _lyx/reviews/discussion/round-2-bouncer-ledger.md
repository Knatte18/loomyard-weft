---
round: 2
ledger:
  - key: d2-reader-dollar-expansion-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d1-no-cutoff-run-plan-not-listed
    rounds: [1]
    status: resolved
  - key: d1-no-cutoff-untimestamped-transcript
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: support-log-question-ledger-order
    rounds: [1]
    status: resolved
  - key: d2-dollar-rule-follower-enumeration
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-redirection-wording-overstated
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 2 maps F1 to the D2 dollar-expansion key (reopened: ANSI-C `$'…'` quoting defeats the command-wide scan), F2 to the new follower-enumeration key, F3 to the untimestamped-transcript key (now also covering the unpinned double-quote and special-parameter dollar cases) and F4 to the new redirection-wording key.
The plan-selection and Q7 ordering keys are resolved: the review raises neither and affirms D1's bound.
F1 and F2 gate.
