---
round: 3
ledger:
  - key: card20-coverage-citation-unsupported
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: card17-orch-stencil-check-outcome-unstated
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: MEDIUM
  - key: card21-watcher-table-underspecified
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: card20-skillunknown-live-path-dropped
    rounds: [2]
    status: resolved
  - key: card21-startup-fold-row-shape-missing
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: card20-fakes-target-and-impact-inaccurate
    rounds: [2]
    status: resolved
  - key: card17-changed-facts-unpinned
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
---

Round 3 raised five findings.
Card 20's coverage citation is now a false justification for keeping `TestRunner_ContextTokens_Delegates` (F5, LOW).
Card 17 still has an unchecked resume step, now the goto line of the escalations section (F1, MEDIUM), and its After landing claim pins nothing that changed (F4, LOW).
Card 21's watcher table still leaves its row shape to the writer (F3, MEDIUM), and the startup table cannot express both elapsed bounds with one zero value (F2, MEDIUM).
The round-2 keys for card 20's dropped `SkillUnknown` live path and its `fakes_test.go` target no longer appear and are resolved.
