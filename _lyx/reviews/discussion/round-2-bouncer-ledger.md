---
round: 2
ledger:
  - key: settler-rule-2-shadows-orch-settles
    rounds: [1]
    status: resolved
  - key: settle-record-reading-outside-allowlist
    rounds: [1]
    status: resolved
  - key: sleep-subtraction-vs-phase-sum
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: formatseconds-drops-seconds-from-one-day
    rounds: [1]
    status: resolved
  - key: partial-runs-and-prewindow-buckets-open
    rounds: [1]
    status: resolved
  - key: driver-resume-gets-no-label
    rounds: [1]
    status: resolved
  - key: rejected-alternative-in-settler-rationale
    rounds: [1]
    status: resolved
  - key: settle-record-to-stop-join-unstated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: non-go-rule-settle-records-unclassified
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings; round 2 raised three, and six of the seven round-1 keys were not raised again.
The sleep key came back at design class: sleep inside phase and bucket time contradicts the tokencount interval rule the Notes cite.
Two keys are new this round: the missing join from settle records to convergence stops, and the unclassified tackler, arbiter and parent settle records.
