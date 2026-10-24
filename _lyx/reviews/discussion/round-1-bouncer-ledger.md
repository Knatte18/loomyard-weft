---
round: 1
ledger:
  - key: d8-guard-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d13-resolveparent-misdirects-finalize
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pattern-pair-parent-text-too-broad
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: beam-orch-undefined-at-landing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dirty-beam-halts-member-landing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-way-forward-dead-end
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: promote-startable-window
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parent-beam-misreads-spawned-members
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: rename-live-beam-strands-members
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d14-owner-setter-kind-change
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-dependency-ban-vs-d10
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-not-derived-per-decision
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d13-runless-notifyprime-callers
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: project-change-splits-parent-readers
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d14-cleanup-sentence-outside-scope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: auto-mode-assumptions-restate
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first ledger; it holds the sixteen findings of the first review, all open.
The gating cluster is the beam design's boundaries: D8's widened guard, D13's `ResolveParent` redefinition against Finalize's parent-records commit, and the declared `PATTERN-pair-parent` text that the design's own origin-record reads contradict.
The MEDIUM design findings are lifecycle gaps: a beam's orch at landing, a dirty beam at member landing, D7 and D9 ordering, promote's startable window, and rename of a live beam.
The LOW and NIT findings are local consistency and acceptance-line corrections.
