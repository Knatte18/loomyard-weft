---
round: 1
ledger:
  - key: file-return-leaves-stale-store-and-parts
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: codex-cache-write-risk-settled-by-repo
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: second-price-table-beside-engine-pricer
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: part-records-no-path-into-board-store
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: auto-approve-key-read-time-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: approval-survives-return-to-raddle-write
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-cold-proposal-in-auto-mode
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-waiting-notice-after-return
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: entry-done-mid-run-hides-later-halts
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: sketch-promote-bound-breaks-on-label-drop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: gone-from-board-read-as-landed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pattern-changes-no-coverage-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cost-fit-ignores-started-discussed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: runs-without-size-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: recheck-misses-relabelled-entries
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: no-done-marker-for-step-3
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: uncited-facts-and-notify-on-land-exception
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: raddle-store-route-to-origin
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: usage-record-pattern-text-overreaches
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stencil-requirement-filed-as-open-risk
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first judged round and has no prior ledger, so every finding enters as open.
Review IDs map to keys in review order: F1 to `file-return-leaves-stale-store-and-parts`, F2 to `codex-cache-write-risk-settled-by-repo`, F3 to `second-price-table-beside-engine-pricer`, F4 to `part-records-no-path-into-board-store`, F5 to `auto-approve-key-read-time-unbounded`, F6 to `approval-survives-return-to-raddle-write`, F7 to `no-cold-proposal-in-auto-mode`, F8 to `no-waiting-notice-after-return`, F9 to `entry-done-mid-run-hides-later-halts`, F10 to `sketch-promote-bound-breaks-on-label-drop`, F11 to `gone-from-board-read-as-landed`, F12 to `pattern-changes-no-coverage-check`, F13 to `cost-fit-ignores-started-discussed`, F14 to `runs-without-size-record`, F15 to `recheck-misses-relabelled-entries`, F16 to `no-done-marker-for-step-3`, F17 to `uncited-facts-and-notify-on-land-exception`, F18 to `raddle-store-route-to-origin`, F19 to `usage-record-pattern-text-overreaches`, F20 to `stencil-requirement-filed-as-open-risk`.

The gating cluster is the Raddle-File and return-to-Raddle-Write interaction (F1, F4, F6, F8, F9, F16, F18), the approval mode (F5, F7), the cost design (F2, F3, F13, F14), and two bound breaks (F10, F11).
Fixes for the design-wrong keys change decisions and need a fresh review in round 2.
