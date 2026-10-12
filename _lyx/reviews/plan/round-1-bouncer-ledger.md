---
round: 1
ledger:
  - key: spawn-failure-skips-fallback
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: notice-truncates-report-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: session-start-stop-guards-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: pralign-call-branches-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: overlap-escalate-question-empty
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: notepath-darn-wording-card-ref
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: shedrun-alignment-paths-granularity
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 is the first round, so there is no earlier ledger.
Two gating design findings at MEDIUM stand: a spawn failure that bypasses the fallback to today's delivery path, and a fallback notice whose 200-rune cut drops the alignment report path.
Two scope findings at MEDIUM ask for test rows reaching unreached guard branches in the session and `PRAlign.Call`.
Three LOW findings cover an overlap escalation with no question, an inconsistent `NotePath` description with a wrong card reference, and the granularity of the `shedrun` alignment path symbols.
