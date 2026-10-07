---
round: 3
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
    status: resolved
  - key: card7-startgated-loses-result
    rounds: [2]
    status: resolved
  - key: runnershuttle-adapter-untested
    rounds: [2]
    status: resolved
  - key: resolvereview-refusals-no-way-forward
    rounds: [2]
    status: resolved
  - key: producer-removal-failures-no-way-forward
    rounds: [2]
    status: resolved
  - key: burlermarker-path-outside-root-refusal
    rounds: [2]
    status: resolved
  - key: card2-pathrules-row-omitted
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: card7-overview-execution-stack-stale
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: card7-halts-no-refusal-standing
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: card7-reviewer-start-failure-unspecified
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: card3-impactsummary-not-conclusion
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 and 2 raised fifteen findings between them; none reappears in round 3, so all are carried as resolved.
Round 3 raised five new findings (F1 to F5 in order of the keys above).
The only gating one is `card2-pathrules-row-omitted`, a MEDIUM design finding about a missing path-rules guard row and a Shared Decisions trigger that would not catch it.
The rest are LOW scope or decision gaps in card 7 and a NIT on card 3's `ImpactSummary`.
Severity is falling round over round (BLOCKING, BLOCKING, MEDIUM).
