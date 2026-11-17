---
round: 2
ledger:
  - key: round-checks-inside-strict-parse
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: origin-parent-exemption-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fixerless-round-indistinguishable
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: carries-bypasses-bound
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-answered-decision-reescalates
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fix-check-cap-behaviour-unstated
    rounds: [1]
    status: resolved
  - key: post-fix-target-id-not-recorded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stalemate-vs-softcap-autoaccept
    rounds: [1]
    status: resolved
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: round-check-context-unreachable
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ruling-disposition-undefined
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: decision-escalation-after-judge
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-bound-restates-regression
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 2 raised 11 findings, F1 through F11 in review order.
F1 maps to `parent-answered-decision-reescalates`, F4 to `post-fix-target-id-not-recorded`, F5 to `origin-parent-exemption-unbounded`,
F7 to `fixerless-round-indistinguishable`, F8 to `round-checks-inside-strict-parse`, and F9 to `carries-bypasses-bound`.
F2, F3, F6, F10 and F11 are new keys.
Round 1 keys the round 2 review does not raise again are resolved: the cap-behaviour, stalemate, regression-in-untouched-file, D2 completeness, amendment, decision-exclusion and D3 acceptance keys.
The BLOCKING F1 is the open core: the parent's answer to a decision-class finding has no per-finding channel.
