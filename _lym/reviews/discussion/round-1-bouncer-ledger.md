---
round: 1
ledger:
  - key: fresh-launch-picks-before-pre-change-resume
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: empty-accounts-list-still-picks
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-resume-search-narrower-than-d6
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: resume-record-write-no-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: status-line-verb-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: opened-sessions-uncovered
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reed-relaunch-stale-directory
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-known-directories-lose-delisted-and-landed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-step4-ranking-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: record-account-name-drift
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: macos-candidates-unbounded-windows-unsourced
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: open-risk-already-settled
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, none carried from an earlier round.
The blockers are the launch-order hole that refuses pre-change sessions (F1) and the missing empty-list gate (F2).
The MEDIUM design findings cover the resume write lock, the status line verb's bounds, opened sessions, reed relaunch, and D6's known set.
The rest are LOW or NIT clean-ups.
