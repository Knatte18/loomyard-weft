---
round: 3
ledger:
  - key: batten-restep-reopens-landing-halts
    rounds: [1]
    status: resolved
  - key: arbiterstop-admits-landing-halts
    rounds: [1]
    status: resolved
  - key: fixer-failure-exits-unprefixed
    rounds: [1]
    status: resolved
  - key: finalize-fix-cap-size-not-kind
    rounds: [1]
    status: resolved
  - key: overcap-reroute-reaches-no-review-at-zero-budget
    rounds: [1]
    status: resolved
  - key: absent-aligned-tip-skips-finalize-merge
    rounds: [1]
    status: resolved
  - key: overlap-test-on-post-merge-head
    rounds: [1]
    status: resolved
  - key: darn-large-merge-reroute-hides-overlap
    rounds: [1]
    status: resolved
  - key: finalize-escalate-verdict-no-outcome
    rounds: [1]
    status: resolved
  - key: idle-backstop-releases-live-finalize-lock
    rounds: [1]
    status: resolved
  - key: deploy-loses-reviewed-code-bound
    rounds: [1]
    status: resolved
  - key: fixed-verdict-advances-aligned-tip-unchecked
    rounds: [1]
    status: resolved
  - key: lock-bound-omits-retry-pass
    rounds: [1]
    status: resolved
  - key: d14-single-table-and-warn-outlet
    rounds: [1]
    status: resolved
  - key: reviewed-tip-file-survives-d4
    rounds: [1]
    status: resolved
  - key: skipped-gate-effect-on-failure-record
    rounds: [1]
    status: resolved
  - key: lock-free-merge-in-criterion-unqualified
    rounds: [1]
    status: resolved
  - key: parent-pair-stucks-dropped-from-merge-in
    rounds: [1]
    status: resolved
  - key: open-risks-restate-d10-and-cost-figure
    rounds: [1]
    status: resolved
  - key: loop-idle-release-frees-finalize-lock
    rounds: [2]
    status: resolved
  - key: d2-upstream-tip-merge-verb-picks-own-tip
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: llm-driver-relaunches-landing-halts
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d14-rewrites-seedonly-profiles
    rounds: [2]
    status: resolved
  - key: finalize-kind-cap-go-only
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d13-deploy-bound-false-at-zero-review-budget
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-pr-review-rests-on-nonexistent-producer
    rounds: [2]
    status: resolved
  - key: declared-recipe-owns-failures-omits-reroute
    rounds: [2]
    status: resolved
  - key: reroute-cap-handback-catches-webster
    rounds: [2]
    status: resolved
  - key: darn-finalize-loses-tree-scoped-command
    rounds: [2]
    status: resolved
  - key: fixer-caps-count-different-units
    rounds: [2]
    status: resolved
  - key: no-review-halt-keys-on-literal-budget
    rounds: [2]
    status: resolved
  - key: finalize-lock-other-release-arms
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-new-engine-rationale-false-premise
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised 19 findings and round 2 raised 12, with a BLOCKING lock-release finding at the top of round 2.
Round 3 raised 6 findings with no BLOCKING one.
The round-2 loop idle-release finding is resolved: both named release paths are guarded and confirmed.
A related finding stays open under a new key, because the batten `ownedElsewhere` and owner-pause arms still release the lock under a Finalize fixer, and Finalize never re-checks the record before landing.
The round-2 D14, D7 producer and four consistency or scope keys are not raised in round 3 and are carried as resolved; D14's read-time migration was explicitly rejected as a finding.
Four earlier keys are still open.
Two of them are gating design findings: the LLM driver's landing-halt exclusion (no recognisable signal, two further re-step sites) and the budget-0 deploy bound (Merge-In fixer omitted).
The other two are `decision` class: how a given SHA reaches the Merge-In conflict session, and where Finalize reads the hub language for the kind cap.
D7's new-engine rationale carries a LOW false-premise finding about `CommandProducer`.
