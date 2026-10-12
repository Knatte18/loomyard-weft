---
round: 2
ledger:
  - key: spawn-failure-skips-fallback
    rounds: [1]
    status: resolved
  - key: notice-truncates-report-path
    rounds: [1]
    status: resolved
  - key: session-start-stop-guards-untested
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
  - key: pralign-call-branches-untested
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
  - key: overlap-escalate-question-empty
    rounds: [1]
    status: resolved
  - key: notepath-darn-wording-card-ref
    rounds: [1]
    status: resolved
  - key: shedrun-alignment-paths-granularity
    rounds: [1]
    status: resolved
  - key: snapshot-read-failure-no-outcome
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: poll-close-errors-no-outcome
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: note-records-moved-head
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: reject-recorded-before-counted
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: orch-claim-reason-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's two gating design findings are resolved: spawn and begin failures now end in today's delivery path, and the notice cut was found not to drop the report path.
The overlap, `NotePath` and `shedrun` granularity LOW findings did not recur.
Round 2 reopens the two untested-guard scope keys through F4, which adds rows for the remaining guard branches in cards 6 and 10.
Round 2's new keys are two gating design MEDIUMs (snapshot read failure, Poll and Close error outcomes), a consistency MEDIUM on the moved-head note, and two LOWs on reject ordering and a stale orch claim reason.
