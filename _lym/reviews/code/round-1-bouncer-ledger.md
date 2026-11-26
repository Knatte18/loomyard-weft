---
round: 1
ledger:
  - key: client-row-field-count-false-unattached-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: client-name-exact-match-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: client-row-cr-trim-uncovered
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, with no prior ledger.
The review approved the diff and raised three findings below the gating line, all in `clientAgentPaneVia` or its test:

- `client-row-field-count-false-unattached-error` (F1): a malformed row for the clicking client is reported as no such client attached.
- `client-name-exact-match-unpinned` (F2): the test's row order lets a prefix match pass.
- `client-row-cr-trim-uncovered` (F3): no test row ends in CRLF.

Each finding's fix text fixes the result, so each is `mechanical`.
