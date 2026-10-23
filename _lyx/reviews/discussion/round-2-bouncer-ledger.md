---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-upstream-tip-merge-verb-picks-own-tip
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: llm-driver-relaunches-landing-halts
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d14-rewrites-seedonly-profiles
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-kind-cap-go-only
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d13-deploy-bound-false-at-zero-review-budget
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-pr-review-rests-on-nonexistent-producer
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: declared-recipe-owns-failures-omits-reroute
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: reroute-cap-handback-catches-webster
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: darn-finalize-loses-tree-scoped-command
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fixer-caps-count-different-units
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-review-halt-keys-on-literal-budget
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 19 findings, led by a BLOCKING batten re-step that undid the landing-halt caps.
Round 2 raised 12 findings and none repeat a round-1 key, so every round-1 key is carried as resolved.
Round 2's BLOCKING finding is the loop's own idle-release hook freeing the parent lock mid-Finalize, the path round 1's batten-only backstop fix left open.
Six design findings at MEDIUM follow: the merge-in verb's tip choice, the LLM driver re-launch, the profiles rewrite, the Go-only kind cap, the deploy bound at review budget 0, and the missing switched producer.
The rest are consistency and scope corrections to settle with local edits.
