---
round: 1
ledger:
  - key: seen-server-ended-criterion-uncovered
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: session-filter-goos-split-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: windows-sources-nothing-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: redundant-boot-validation-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: redundant-testfill-tmux-conf-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: impact-summary-site-count
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised six findings, none gating.
F1 is the one MEDIUM: the seen-server-ended acceptance criterion maps to a differing-identity row no planned subtest holds.
F2 and F3 are missing coverage for the session filter's GOOS split and for Windows sourcing nothing.
F4 and F5 are LOW test-economy redundancies in cards 2 and 1.
F6 is a NIT on the ImpactSummary site count in card 4.
