---
round: 3
ledger:
  - key: card3-auto-path-refusals-still-name-retired-verbs
    rounds: [1]
    status: resolved
  - key: card14-read-only-classifier-not-wired-in-loomcli
    rounds: [1, 2]
    status: resolved
  - key: card21-omits-shellwaitactive-and-run-constructors
    rounds: [1, 2]
    status: resolved
  - key: card19-derived-text-set-incomplete
    rounds: [1, 2]
    status: resolved
  - key: card11-sourcepath-unresolvable-use-card-text
    rounds: [1]
    status: resolved
  - key: card2-test-rows-have-no-seam
    rounds: [1]
    status: resolved
  - key: card17-tsetenv-in-parallel-test
    rounds: [1]
    status: resolved
  - key: card16-lands-before-card17
    rounds: [1]
    status: resolved
  - key: card4-warning-path-to-summary-unnamed
    rounds: [1, 2]
    status: resolved
  - key: card5-restep-claim-false-for-save-restamp
    rounds: [1]
    status: resolved
  - key: card15-nil-classifier-rule-missing
    rounds: [1]
    status: resolved
  - key: card23-recoveryretry-git-side-untested
    rounds: [1]
    status: resolved
  - key: card9-fixer-path-untested
    rounds: [1]
    status: resolved
  - key: cards13-16-reader-set-drift
    rounds: [1, 2]
    status: resolved
  - key: card4-warning-dropped-on-some-stuck-exits
    rounds: [1, 2]
    status: resolved
  - key: card6-test-duplicates-card7
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: LOW
  - key: card23-rebaseline-keeps-count-unasserted
    rounds: [1]
    status: resolved
  - key: card16-acceptance-clauses-unasserted
    rounds: [1]
    status: resolved
  - key: cards15-16-attack-surface-bound-unstated
    rounds: [1]
    status: resolved
  - key: card23-head-read-failure-undecided
    rounds: [1]
    status: resolved
  - key: cards3-6-missing-doc-go-edit
    rounds: [1]
    status: resolved
  - key: thin-test-rows-cards-13-11-17-21
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: MEDIUM
  - key: perishable-tallies-and-stray-targets
    rounds: [1]
    status: resolved
  - key: card20-recoverbatch-test-not-in-edit
    rounds: [2]
    status: resolved
  - key: card21-no-gate-arrival-test-misses-failed-send
    rounds: [2]
    status: resolved
  - key: card21-ended-shells-logged-where-no-removal-happens
    rounds: [2]
    status: resolved
  - key: card24-claims-test-not-in-edit
    rounds: [2]
    status: resolved
  - key: card3-log-rows-meet-process-global-logger
    rounds: [2]
    status: resolved
  - key: card21-waitingtasks-cleared-before-finalize
    rounds: [2]
    status: resolved
  - key: card11-widens-planparser-dependency-tree
    rounds: [2]
    status: resolved
  - key: card8-unreadable-record-renders-none
    rounds: [2]
    status: resolved
  - key: card4-merriambase-failure-escapes-stuck
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
  - key: card14-classifier-rows-miss-glob-and-tilde
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
  - key: card4-widens-errautorebaseline
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: card3-no-partition-branch-names-manual-verbs
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: card22-started-fresh-resumed-session-open
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: card24-two-head-reads-in-recoverspawn
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: card2-single-batch-check-unpinned
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: card3-existing-rebaseline-row-asserts-reworded-text
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: card18-guard-wayforward-and-help-rows-unasserted
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: card4-save-failure-and-log-once-rows-underspecified
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: card24-no-nondead-terminal-prior-row
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: card1-runstdin-second-checked-form
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
  - key: card18-fabric-go-header-not-target
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised 23 findings, round 2 raised 32 and round 3 raised 15 with no BLOCKING.
Every round 1 and round 2 key without a round 3 counterpart is carried as resolved, including the round 2 BLOCKING `card20-recoverbatch-test-not-in-edit` and the gating `design` MEDIUMs on card 21.

Two older keys recur in round 3.
`thin-test-rows-cards-13-11-17-21` recurs as F2 (card 17 slotted branch), at `scope` MEDIUM.
`card6-test-duplicates-card7` recurs as F10, now filed as `design` LOW.

The other round 3 findings are new keys, mostly test-row gaps and wording drift on cards 2, 3, 4, 14, 18, 22 and 24; the three MEDIUMs are all `scope`.
