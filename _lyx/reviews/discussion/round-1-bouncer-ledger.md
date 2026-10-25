---
round: 1
ledger:
  - key: d7-runproc-end-kills-passes
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: prime-anchor-change-strands-other-machines
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-recovery-states-indistinguishable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reconcile-premise-false-in-recovery
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: uncommitted-move-undo-unspecified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ended-dobbies-not-restarted-after-kill
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: automatic-dobby-starts-break-lock-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: child-run-landings-during-move
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pull-refusal-incoming-tip-undefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reconcile-marker-commits-never-pushed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: census-blind-to-unfetched-branches
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: empty-marker-census-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-fabric-anchor-breaks-migrated-hub
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: fresh-bind-probe-cannot-see-main-weft
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-marker-refusal-after-step-1
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-rerun-skip-underspecified
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised sixteen findings against the discussion pair, all still open.
The core is D7: its step order (F1, F15), its recovery (F3, F4, F5, F6), its lock bound (F7) and its refusals (F8) each have a gap, and the prime case (F2) has no design for other machines.
D10's pull refusal (F9) and D8's marker census (F10, F11, F12, F13) are the other clusters; D4's probe (F14) and D6's re-run (F16) are small.
Finding IDs map to keys in review order: F1 `d7-runproc-end-kills-passes`, F2 `prime-anchor-change-strands-other-machines`, F3 `d7-recovery-states-indistinguishable`, F4 `reconcile-premise-false-in-recovery`, F5 `uncommitted-move-undo-unspecified`, F6 `ended-dobbies-not-restarted-after-kill`, F7 `automatic-dobby-starts-break-lock-bound`, F8 `child-run-landings-during-move`, F9 `pull-refusal-incoming-tip-undefined`, F10 `reconcile-marker-commits-never-pushed`, F11 `census-blind-to-unfetched-branches`, F12 `empty-marker-census-unstated`, F13 `stale-fabric-anchor-breaks-migrated-hub`, F14 `fresh-bind-probe-cannot-see-main-weft`, F15 `empty-marker-refusal-after-step-1`, F16 `d6-rerun-skip-underspecified`.
