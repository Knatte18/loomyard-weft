---
round: 2
ledger:
  - key: test-budget-rows-not-raised
    rounds: [1, 2]
    status: open
    class: scope
    severity: BLOCKING
  - key: card6-tmux-tier-repoName-literals
    rounds: [1]
    status: resolved
  - key: card15-spawn-integration-callers
    rounds: [1, 2]
    status: open
    class: scope
    severity: BLOCKING
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
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: retired-status-line-stale-references
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card2-validatesegmentcolors-test-duplicates-bootvalidation
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: card4-switch-client-optional-verb-doc-early
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised ten findings, three BLOCKING scope gaps and seven LOW.
Round 2 does not re-raise the seven LOW findings or the card 6 `RepoName` gap, so they are resolved.
Both remaining round-1 BLOCKING keys recur in new form: card 15 still misses a caller of `Spawn` (`internal/ideengine/menu.go`), and the test-budget gap now appears for card 14's `orchengine` row.
New in round 2: card 5's `status 2` breaks existing tmux-tier height assertions (MEDIUM), stale retired-status-line references in M19 and `yamlengine.md` (LOW), a redundant `validateSegmentColors` test plan in card 2 (LOW), and card 4's early optional-verb wording (NIT).
