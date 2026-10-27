---
round: 1
ledger:
  - key: adopt-seed-reaches-every-migrated-document
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: no-test-reaches-top-of-loop-worktree-gone-exit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: trace-test-row-breaks-hub-for-later-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card7-extends-doc-it-rewrites
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: beam-status-readcheck-writes-scratch-dir
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refused-tier-leaves-running-set
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: post-tick-record-read-recreates-stub
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: fatal-stop-line-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: trace-test-rows-recover-existing-test-inputs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-keys-table-and-adopt-scratch-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: empty-name-guards-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: busy-slots-cannot-end-waiting-gate
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first ledger.
The one gating finding is the unreached top-of-iteration exit in card 8's `TestRun_WorktreeGoneExits`.
The other MEDIUM, the adopt seed reaching every `MigrateConfig` caller, is a `decision`-class question about card 11's scope and needs a design choice from the author.
The rest are local corrections to test rows, doc lists and text.
