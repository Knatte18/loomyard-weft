---
round: 1
ledger:
  - key: open-load-turn-skips-and-types-prompt
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hold-behaviors-lack-pinning-rows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: remaining-hold-paths-uncovered-duplicate-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-prose-doc-run-notifier-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-entries-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: model-rejected-end-and-parent-notice-unrecorded
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings: F1 to F6, in the order of the keys above.
F1 is the only design-wrong finding: a load turn still open at `loadTurnOpenBound`, or never seen to start, still ends as a skip and types the prompt.
F2 and F3 ask for rows in `TestStartup_LoadTurnAPIError`.
F4 and F5 are stale prose and incomplete pattern entries.
F6 is a scope finding the fixer must defer to the run's parent rather than revert.
