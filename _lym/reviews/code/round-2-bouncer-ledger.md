---
round: 2
ledger:
  - key: dobbyruns-unreadable-landed-commit-aborts-listing
    rounds: [1]
    status: resolved
  - key: first-try-record-landing-unasserted
    rounds: [1]
    status: resolved
  - key: landnotice-requested-refresh-and-failed-read-unpinned
    rounds: [1]
    status: resolved
  - key: notify-on-land-revised-date-and-replay-unasserted
    rounds: [1]
    status: resolved
  - key: landing-notice-deps-wirings-unpinned
    rounds: [1]
    status: resolved
---

Round 1 raised five `design` findings: two MEDIUM, three LOW.
Round 2 raised none, and the reviewer confirmed each earlier concern is fixed and pinned by a named test.
All five keys are resolved.
