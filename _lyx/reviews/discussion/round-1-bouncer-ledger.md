---
round: 1
ledger:
  - key: settler-rule-2-shadows-orch-settles
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: settle-record-reading-outside-allowlist
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: sleep-subtraction-vs-phase-sum
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: formatseconds-drops-seconds-from-one-day
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: partial-runs-and-prewindow-buckets-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: driver-resume-gets-no-label
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: rejected-alternative-in-settler-rationale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, none carried from earlier rounds.
The gating one is the settler classification: rule 2 matches any Go driver settle record, so records for moved, busy-exit, running-exit and hand-over make an orch-settled stop read as `shed`.
Three MEDIUM consistency findings follow: the `reportengine` allowlist growth, sleep subtraction against the phase-sum criterion, and `FormatSeconds` at a day and over.
The rest are an open window choice, the driver resume label and a rejected alternative that belongs in the support log.
