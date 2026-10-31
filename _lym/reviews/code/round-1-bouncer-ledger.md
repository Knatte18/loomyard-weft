---
round: 1
ledger:
  - key: dobbyruns-unreadable-landed-commit-aborts-listing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: first-try-record-landing-unasserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: landnotice-requested-refresh-and-failed-read-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: notify-on-land-revised-date-and-replay-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: landing-notice-deps-wirings-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all class `design`.
Two are MEDIUM and gate: an unreadable landed-commit file aborting the run listing, and the missing first-try `recordLanding` assertion.
Three are LOW test-coverage gaps: the `Requested` refresh and failed board read, the revised date and replay of `notify_on_land`, and the loomcli and beamcli deps wirings.
Every fix is a local correction whose result the finding text fixes.
