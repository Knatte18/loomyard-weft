---
round: 3
ledger:
  - key: new-error-paths-no-way-forward-no-spec-row
    rounds: [1, 2]
    status: resolved
  - key: transient-refusals-without-spec-row
    rounds: [1]
    status: resolved
  - key: migrateconfig-orientation-insert-index
    rounds: [1, 2]
    status: resolved
  - key: run-help-dash-reads-as-flag
    rounds: [1]
    status: resolved
  - key: recover-batch-uncommitted-refusal-untested
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: rebaseline-frame-rerun-drops-card-flag
    rounds: [2]
    status: resolved
  - key: standalone-reset-uncommitted-only-for-start
    rounds: [3]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised four findings and round 2 four more; the gating ones are resolved.
Round 3 found nothing BLOCKING and nothing `design`.
The `recover-batch` refusal is now tested for a failing dirty-path read, and F2 narrows the remaining gap to the run-writes audit cause, missing from the spec row and the tests.
F1 is new: standalone resets to targets other than start list no uncommitted paths, against card 15's wording.
Both open entries are LOW and non-gating.
