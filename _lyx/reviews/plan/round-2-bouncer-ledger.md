---
round: 2
ledger:
  - key: card6-breaks-burlerengine-template-test
    rounds: [1]
    status: resolved
  - key: card3-config-refusals-no-spec-row
    rounds: [1]
    status: resolved
  - key: strandremover-hidden-in-engine-card
    rounds: [1]
    status: resolved
  - key: card6-no-handoff-bound
    rounds: [1]
    status: resolved
  - key: card1-probegated-retests-attach
    rounds: [1]
    status: resolved
  - key: step3-header-names-deleted-orchestrator
    rounds: [1]
    status: resolved
  - key: overview-doc-not-updated
    rounds: [1]
    status: resolved
  - key: card4-untested-error-exits
    rounds: [1]
    status: resolved
  - key: card7-donegone-row-cannot-assert-removal
    rounds: [1]
    status: resolved
  - key: card-targets-omit-forced-files
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: card7-startgated-loses-result
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: runnershuttle-adapter-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: resolvereview-refusals-no-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: producer-removal-failures-no-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: burlermarker-path-outside-root-refusal
    rounds: [2]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised nine findings; none reappears in round 2, so all are carried as resolved.
Round 2 raised six new findings.
The gating ones are the BLOCKING method gap in card target lists (`card-targets-omit-forced-files`, F1) and the MEDIUM `StartGated` result loss in card 7 (`card7-startgated-loses-result`, F2).
`card-targets-omit-forced-files` generalises the round-1 scope findings `card6-breaks-burlerengine-template-test` and `overview-doc-not-updated`.
The non-gating findings are the untested `RunnerShuttle` adapter (F3), two refusals lacking a way-forward clause or spec row (F4, F5) and the `burlermarker.Path` outside-root refusal (F6).
