---
round: 1
ledger:
  - key: test-budget-rows-not-raised
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card6-tmux-tier-repoName-literals
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card15-spawn-integration-callers
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card3-reassert-not-reached-on-chain-degrades
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card13-impactsummary-tally
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card13-roles-go-not-a-target
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card3-paneoptions-argv-table-overlap
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card8-reapply-test-duplicates-skipfocus
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card7-view-click-unbounded-state
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card10-seed-mtime-start-time-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

Round 1 is the first review of the plan.
Three scope gaps block: the test-budget file is named by no card, card 6 leaves tmux-tier tests that set the removed `Geometry.RepoName`, and card 15 changes `ideengine.Spawn` without its integration callers.
Seven LOW findings concern card 3's re-assert placement, card 13's tally and uncovered `roles.go`, two redundant test plans in cards 3 and 8, and unstated bounds in cards 7 and 10.
