---
round: 3
ledger:
  - key: test-budget-rows-not-raised
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card6-tmux-tier-repoName-literals
    rounds: [1]
    status: resolved
  - key: card15-spawn-integration-callers
    rounds: [1, 2]
    status: resolved
  - key: card3-reassert-not-reached-on-chain-degrades
    rounds: [1]
    status: resolved
  - key: card13-impactsummary-tally
    rounds: [1]
    status: resolved
  - key: card13-roles-go-not-a-target
    rounds: [1]
    status: resolved
  - key: card3-paneoptions-argv-table-overlap
    rounds: [1]
    status: resolved
  - key: card8-reapply-test-duplicates-skipfocus
    rounds: [1]
    status: resolved
  - key: card7-view-click-unbounded-state
    rounds: [1]
    status: resolved
  - key: card10-seed-mtime-start-time-unbounded
    rounds: [1]
    status: resolved
  - key: card5-status2-breaks-tmux-height-tests
    rounds: [2, 3]
    status: open
    class: scope
    severity: MEDIUM
  - key: retired-status-line-stale-references
    rounds: [2]
    status: resolved
  - key: card2-validatesegmentcolors-test-duplicates-bootvalidation
    rounds: [2]
    status: resolved
  - key: card4-switch-client-optional-verb-doc-early
    rounds: [2]
    status: resolved
  - key: attach-chain-shape-tests-unlisted
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
  - key: card3-strandlabel-table-overlap
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 and 2 raised BLOCKING scope gaps; round 3 has none and no design-class finding.
`card15-spawn-integration-callers` is resolved: round 3 confirmed every `Spawn` caller is in card 15's Edit list.
Round 2's LOW and NIT keys are not raised again, so they are resolved.
`test-budget-rows-not-raised` recurs as F3 (cards 12 and 13, consistency MEDIUM), and `card5-status2-breaks-tmux-height-tests` recurs as F1 (further tmux-tier files, scope MEDIUM).
New in round 3: F2 (attach-chain shape assertions in `attachgeometry_integration_test.go`) and F4 (`strandLabel` table duplicating the launch-fixture test).
