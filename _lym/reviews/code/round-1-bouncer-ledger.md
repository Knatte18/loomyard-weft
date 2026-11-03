---
round: 1
ledger:
  - key: submit-drops-owned-kept-pause
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: switch-failure-paths-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: kept-entry-record-write-failed-unrecorded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: keep-path-notice-no-owned-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: config-load-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all `design`, none BLOCKING.
F1 (`submit-drops-owned-kept-pause`) changes `Submit`'s contract to refuse over a terminal request whose owned entry is live, so its fix needs a fresh review.
F2 asks for `TestWatch` rows pinning the switch-off failure paths.
F3 asks the keep path to record an entry whose owned-record write failed.
F4 and F5 ask for missing test rows.
