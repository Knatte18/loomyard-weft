---
round: 3
ledger:
  - key: d8-guard-unbounded
    rounds: [1, 2]
    status: resolved
  - key: d13-resolveparent-misdirects-finalize
    rounds: [1]
    status: resolved
  - key: pattern-pair-parent-text-too-broad
    rounds: [1]
    status: resolved
  - key: beam-orch-undefined-at-landing
    rounds: [1]
    status: resolved
  - key: dirty-beam-halts-member-landing
    rounds: [1]
    status: resolved
  - key: d7-way-forward-dead-end
    rounds: [1]
    status: resolved
  - key: promote-startable-window
    rounds: [1, 2]
    status: resolved
  - key: parent-beam-misreads-spawned-members
    rounds: [1, 2]
    status: resolved
  - key: rename-live-beam-strands-members
    rounds: [1]
    status: resolved
  - key: d14-owner-setter-kind-change
    rounds: [1]
    status: resolved
  - key: d1-dependency-ban-vs-d10
    rounds: [1]
    status: resolved
  - key: acceptance-not-derived-per-decision
    rounds: [1]
    status: resolved
  - key: d13-runless-notifyprime-callers
    rounds: [1]
    status: resolved
  - key: project-change-splits-parent-readers
    rounds: [1]
    status: resolved
  - key: d14-cleanup-sentence-outside-scope
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate
    rounds: [1]
    status: resolved
  - key: beam-land-blocked-by-start-guard
    rounds: [2]
    status: resolved
  - key: dirty-prime-refuses-member-create
    rounds: [2]
    status: resolved
  - key: parentof-found-record-slug-unstated
    rounds: [2]
    status: resolved
  - key: project-layers-as-ready
    rounds: [2]
    status: resolved
  - key: finalize-reroute-drop-unbounded
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-bound-holds-only-on-prime-owner-machine
    rounds: [2]
    status: resolved
  - key: ready-walk-beam-absent-here
    rounds: [2]
    status: resolved
  - key: beam-orch-strand-hardcodes-shortname
    rounds: [2]
    status: resolved
  - key: d13-ready-notice-reaches-barred-prime-orch
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d10-weft-reset-outside-destroy-chokepoint
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-refusal-rule-stated-twice
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised sixteen findings and round 2 eleven; most of the round 1 set resolved by round 2, and round 2's beam-landing cluster (start guard, dirty prime, `ParentOf` reading, project layering) did not recur in round 3.

Round 3 raised four findings, none BLOCKING.
The one design finding is D10 step 4: the weft reset has no stated route through `internal/fabricengine/destroy.go` and no stated treatment of uncommitted weft content.
The rest are bounds and wording: D11's bound understates the dropped reroute (a continuation of the round 2 key), D13's ready notice reaches a prime orch that D8 bars from the task, and D5 states one rule twice.
