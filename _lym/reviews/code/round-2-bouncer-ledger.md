---
round: 2
ledger:
  - key: raddle-start-clears-before-owner-check
    rounds: [1]
    status: resolved
  - key: clear-motif-run-closure-untested
    rounds: [1]
    status: resolved
  - key: raddle-file-exemption-ignores-ended-how
    rounds: [1]
    status: resolved
  - key: start-clears-recipe-resolution-differs
    rounds: [1]
    status: resolved
  - key: sketch-finish-failure-paths-untested
    rounds: [1]
    status: resolved
  - key: dobby-done-run-refusal-doc-exception
    rounds: [1]
    status: resolved
  - key: loomcli-doc-ended-origin-raddle-file
    rounds: [1]
    status: resolved
  - key: refused-sketch-notice-upsert-payload
    rounds: [1]
    status: resolved
  - key: refused-sketch-notice-set-deps-not-runnable
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: battencli-doc-guard-removal-order
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dobby-tasks-unreadable-start-judgment-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1's eight keys are resolved: round 2 raises none of them, and its settled focus questions confirm the fixes.
Round 2 raises three new keys.
F1 (`refused-sketch-notice-set-deps-not-runnable`) is a bare `set-deps` in the refused-Sketch notice.
F2 (`battencli-doc-guard-removal-order`) is a package doc that states the guard and removal in the wrong order.
F3 (`dobby-tasks-unreadable-start-judgment-untested`) is a missing test row for the hold on an unreadable start judgment.
All three have fix text that fixes the result, so each is mechanical.
