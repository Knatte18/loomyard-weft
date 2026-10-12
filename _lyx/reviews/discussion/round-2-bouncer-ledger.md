---
round: 2
ledger:
  - key: d7-row-time-measured-from-wrong-end
    rounds: [1]
    status: resolved
  - key: no-strand-absorbs-free-rows-after-orch-stop
    rounds: [1]
    status: resolved
  - key: notice-log-trigger-misses-held-back-and-dropped
    rounds: [1]
    status: resolved
  - key: anchor-kind-combinations-without-height
    rounds: [1]
    status: resolved
  - key: d2-d7-unstated-state-outcomes
    rounds: [1]
    status: resolved
  - key: lyx-cleanup-risk-is-a-constraint
    rounds: [1]
    status: resolved
  - key: deliberation-narrative-in-decision-record
    rounds: [1]
    status: resolved
  - key: missing-acceptance-criteria
    rounds: [1]
    status: resolved
  - key: d7-halt-wait-attribution
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-halted-run-box-and-clock
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: d8-notice-append-identity
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
---

Round 1 raised eight findings, two of them BLOCKING design findings.
Round 2 did not re-raise any of them; the review confirms the D3 and D1 rows, the D7 stamping rule and the D8 held-back and dropped coverage against the code, so all eight are resolved.

Round 2 raised three new findings.
F1 (`d7-halt-wait-attribution`, MEDIUM design) asks how wall time spent while the run is halted, paused or awaiting is attributed to a row, including across a goto.
F2 (`d7-halted-run-box-and-clock`, LOW decision) asks which box and clock the current row shows in a paused, blocked or awaiting run.
F3 (`d8-notice-append-identity`, MEDIUM decision) asks what identity the notice-log append is keyed on, since "once" holds only within one batten process.
