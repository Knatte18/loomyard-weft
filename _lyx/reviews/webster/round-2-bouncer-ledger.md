---
round: 2
ledger:
  - key: new-error-paths-no-way-forward-no-spec-row
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: transient-refusals-without-spec-row
    rounds: [1]
    status: resolved
  - key: migrateconfig-orientation-insert-index
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
  - key: run-help-dash-reads-as-flag
    rounds: [1]
    status: resolved
  - key: recover-batch-uncommitted-refusal-untested
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: rebaseline-frame-rerun-drops-card-flag
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised four findings.
The round-2 review confirms the fixes for the spec rows and the help text, so those two keys are resolved.
The round-1 refusal gap persists in `Run`'s overview-frame re-read (F2), an unreachable refusal.
A sibling refusal in `recover-batch` has a spec row but no test (F1).
The `MigrateConfig` lowest-index fix holds, but its test pins only one file order (F4).
The rebaseline overview-frame re-run wording omits `--card` (F3).
