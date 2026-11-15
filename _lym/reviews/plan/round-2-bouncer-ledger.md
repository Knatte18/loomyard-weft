---
round: 2
ledger:
  - key: pairteardown-imports-battenrecipe-cycle
    rounds: [1]
    status: resolved
  - key: dobbyengine-allowlist-missing-runfsm
    rounds: [1]
    status: resolved
  - key: board-status-affected-set-incomplete
    rounds: [1]
    status: resolved
  - key: cancelled-branch-read-network-per-tick
    rounds: [1]
    status: resolved
  - key: shed-pause-aside-unlicensed
    rounds: [1]
    status: resolved
  - key: cancel-invisible-before-intent-committed
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: paused-refusal-names-wrong-verb
    rounds: [1]
    status: resolved
  - key: drivable-reached-parent-clause-dead
    rounds: [1]
    status: resolved
  - key: runfsm-order-test-only
    rounds: [1]
    status: resolved
  - key: cancel-marker-prose-survives
    rounds: [1]
    status: resolved
  - key: landed-slug-pause-unliftable
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cancel-act-rule-row-unreachable
    rounds: [1]
    status: resolved
  - key: nil-pairstands-meaning
    rounds: [1]
    status: resolved
  - key: card9-impactsummary-undercount
    rounds: [1]
    status: resolved
  - key: loom-run-step-unasked-no-reason
    rounds: [1]
    status: resolved
  - key: readrun-pastparentmerge-no-pair
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: readrun-tier1-test-package
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-edit-target-member-glyphs
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised fifteen findings, two of them BLOCKING.
Round 2 no longer shows thirteen of them.
Two keys continue in a new form.
`cancel-invisible-before-intent-committed` is now card 11 dropping the cancel-lock check from pass planning (F1, consistency MEDIUM).
`landed-slug-pause-unliftable` is now the Landed refusal naming a read-only check (F5, design LOW).
Three keys are new, all on card 5: `readrun-pastparentmerge-no-pair` (MEDIUM), `readrun-tier1-test-package` and `card5-edit-target-member-glyphs` (LOW).
Each of the five fixes is a local rewording, addition or test-package note, so none changes a decision.
