---
round: 3
ledger:
  - key: round-checks-inside-strict-parse
    rounds: [1, 2]
    status: resolved
  - key: origin-parent-exemption-unbounded
    rounds: [1, 2]
    status: resolved
  - key: fixerless-round-indistinguishable
    rounds: [1, 2]
    status: resolved
  - key: carries-bypasses-bound
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parent-answered-decision-reescalates
    rounds: [1, 2]
    status: resolved
  - key: fix-check-cap-behaviour-unstated
    rounds: [1]
    status: resolved
  - key: post-fix-target-id-not-recorded
    rounds: [1, 2]
    status: resolved
  - key: stalemate-vs-softcap-autoaccept
    rounds: [1, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stalemate-arbiter-eligibility
    rounds: [1]
    status: resolved
  - key: fix-regression-untouched-file-refused
    rounds: [1]
    status: resolved
  - key: d2-completeness-vs-open-risk
    rounds: [1]
    status: resolved
  - key: amendment-rounds-full-ambiguity
    rounds: [1]
    status: resolved
  - key: decision-exclusion-assumes-routing
    rounds: [1]
    status: resolved
  - key: d3-acceptance-omits-repeats
    rounds: [1]
    status: resolved
  - key: answer-baseline-count-breaks-on-replace
    rounds: [2]
    status: resolved
  - key: round-check-context-unreachable
    rounds: [2]
    status: resolved
  - key: ruling-disposition-undefined
    rounds: [2]
    status: resolved
  - key: decision-escalation-after-judge
    rounds: [2]
    status: resolved
  - key: d4-bound-restates-regression
    rounds: [2]
    status: resolved
  - key: routed-fact-unreachable
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: resolved-status-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: validate-review-worktree-resolution
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: title-collision-loop-digest
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 3 raised six findings, all `design`, none BLOCKING.
F2 maps to `carries-bypasses-bound` and F5 to `stalemate-vs-softcap-autoaccept`, both reopened from earlier rounds.
F1, F3, F4 and F6 are new keys: `routed-fact-unreachable`, `resolved-status-unbounded`,
`validate-review-worktree-resolution` and `title-collision-loop-digest`.
Every round 2 key the round 3 review does not raise again is resolved; its rejected list covers the baseline, post-fix id, parent exemption and fixer-less round concerns.
The remaining work is the stalemate check's place against `softCapRule`'s accept and the carries edge's class bound.
