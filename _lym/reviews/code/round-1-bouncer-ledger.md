---
round: 1
ledger:
  - key: edit-hook-shares-events-file
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: friction-note-reinterpreted-as-notice
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: stamp-time-counting-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-offset-persist-drops-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: codex-runs-no-edit-loop-watch
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: notice-content-asserted-against-itself
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: recordedit-open-failure-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-omits-edit-loop
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger exists.
F1 (shared events file) is the one finding that changes design: the fix moves edit records to their own file.
F2 and F5 are parent decisions the fixer leaves in place.
The rest are local test, retry and documentation corrections.
