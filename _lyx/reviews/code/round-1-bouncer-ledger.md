---
round: 1
ledger:
  - key: pattern-shed-stop-chain-stale-for-webster
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: run-decision-record-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: master-none-row-asserts-stencil-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: judges-first-webster-only-marking-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
The `PATTERN.md` entry for the stop chain does not name the Webster arbiter-first exception (F1, BLOCKING).
The `Run` refusal for an unreadable decision record and its pass-through of the record path have no test (F2).
The master `none` row quotes stencil prose (F3).
The Webster-only `JudgesFirst` wiring is not pinned by a test (F4).
Each has a local fix the finding text already fixes.
