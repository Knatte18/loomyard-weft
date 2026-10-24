---
round: 2
ledger:
  - key: d8-guard-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-beam-misreads-spawned-members
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: dirty-prime-refuses-member-create
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parentof-found-record-slug-unstated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: project-layers-as-ready
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-reroute-drop-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-bound-holds-only-on-prime-owner-machine
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: ready-walk-beam-absent-here
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: beam-orch-strand-hardcodes-shortname
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised sixteen findings.
Round 2's fresh review does not repeat thirteen of them, so those are resolved.
Three recur in narrower form: the D8 guard now at the landing's own Finalize agents, the open-member refusal's `ParentBeam` reading, and D10's promote-stop criterion.

Round 2's gating cluster is the beam landing's path: batten's start guard refuses the landing over the existing pair (BLOCKING), a dirty prime refusing a member's create, `ParentOf`'s unstated reading of a found origin record, and projects layering as Ready.
The remaining new findings are LOW bounds and consistency lines, plus one NIT on the strand name's shortname.
