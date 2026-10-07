---
round: 4
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
    status: resolved
  - key: card7-overview-execution-stack-stale
    rounds: [3]
    status: resolved
  - key: card7-halts-no-refusal-standing
    rounds: [3]
    status: resolved
  - key: card7-reviewer-start-failure-unspecified
    rounds: [3]
    status: resolved
  - key: card3-impactsummary-not-conclusion
    rounds: [3]
    status: resolved
  - key: card7-plain-start-error-leaves-reviewer-live
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
  - key: card7-burlercli-cli-go-not-a-target
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
  - key: card2-pathrules-row-relative-review-path
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: cards1-7-impactsummary-exclusivity-claims
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: card8-resume-test-repeats-card7-rows
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: directive-stencils-still-describe-combined-agent
    rounds: [4]
    status: open
    class: scope
    severity: LOW
  - key: card7-marker-io-halts-unplaced
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 to 3 raised twenty findings between them; none reappears in round 4, so all are carried as resolved.
The round 3 findings were each closed or ruled not a defect by the round 4 reviewer's focus outcomes.
Round 4 raised seven new findings (F1 to F7 in order of the keys above), none of them `design`.
Severity and gating fell round over round: the gating count went 2, 2, 1, 0.
The two MEDIUM findings are scope gaps in card 7 (an unpinned plain start error, a missing `cli.go` target).
