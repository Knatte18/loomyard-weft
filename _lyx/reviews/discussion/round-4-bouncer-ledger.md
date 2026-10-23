---
round: 4
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
    status: resolved
  - key: llm-driver-relaunches-landing-halts
    rounds: [2, 3]
    status: resolved
  - key: d14-rewrites-seedonly-profiles
    rounds: [2]
    status: resolved
  - key: finalize-kind-cap-go-only
    rounds: [2, 3]
    status: resolved
  - key: d13-deploy-bound-false-at-zero-review-budget
    rounds: [2, 3, 4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
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
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-new-engine-rationale-false-premise
    rounds: [3]
    status: resolved
  - key: merge-in-budget0-cap-baseline-precedes-merge
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-reroute-cap-halt-names-moving-parent-only
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: fixer-attempt-record-no-reset-rule
    rounds: [4]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 4 raised five findings with no BLOCKING one, down from six in round 3.
The four gating or decision keys open after round 3 (`d2-upstream-tip-merge-verb-picks-own-tip`, `llm-driver-relaunches-landing-halts`, `finalize-kind-cap-go-only`, and the design form of the budget-0 deploy bound) and `d7-new-engine-rationale-false-premise` are not raised in round 4 and are carried as resolved.
The round-3 lock-release key and the budget-0 deploy-bound key come back only as LOW consistency wording corrections: the release causes in D10 and the omitted conflict-session resolution in D13.
The one gating-class finding is new this round: Merge-In's budget-0 size and kind cap measures from a HEAD that precedes the merge.
Two further new keys are non-gating: the Finalize reroute-cap halt's way-forward clause names a moving parent only, and the persisted fixer attempt record has no reset rule.
