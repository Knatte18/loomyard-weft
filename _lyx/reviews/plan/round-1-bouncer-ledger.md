---
round: 1
ledger:
  - key: card8-batten-rows-between-run-shed-no-phase
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card5-resume-from-paused-label-disagrees
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card6-label-shed-test-duplicates-driverdeps
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised three MEDIUM consistency findings.
F1 (key `card8-batten-rows-between-run-shed-no-phase`) is card 8's phase rules leaving batten rows between `Run-Shed` entries without a phase.
F2 (key `card5-resume-from-paused-label-disagrees`) is card 5's plain step recording `ResumedBy` on a resume from paused while `RecordResumer` does not.
F3 (key `card6-label-shed-test-duplicates-driverdeps`) is card 6's new top-level test duplicating `TestBattenDriverDeps`.
Each fix is a local correction the finding text fixes.
