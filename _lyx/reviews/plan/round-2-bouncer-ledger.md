---
round: 2
ledger:
  - key: card3-auto-path-refusals-still-name-retired-verbs
    rounds: [1]
    status: resolved
  - key: card14-read-only-classifier-not-wired-in-loomcli
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card21-omits-shellwaitactive-and-run-constructors
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
  - key: card19-derived-text-set-incomplete
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
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
    status: open
    class: consistency
    severity: MEDIUM
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
    status: open
    class: design
    severity: LOW
  - key: card4-warning-dropped-on-some-stuck-exits
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
  - key: card6-test-duplicates-card7
    rounds: [1, 2]
    status: open
    class: consistency
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
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
  - key: perishable-tallies-and-stray-targets
    rounds: [1]
    status: resolved
  - key: card20-recoverbatch-test-not-in-edit
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: card21-no-gate-arrival-test-misses-failed-send
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: card21-ended-shells-logged-where-no-removal-happens
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: card24-claims-test-not-in-edit
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card3-log-rows-meet-process-global-logger
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card21-waitingtasks-cleared-before-finalize
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: card11-widens-planparser-dependency-tree
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: card8-unreadable-record-renders-none
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised 23 findings; round 2 raised 32, one BLOCKING.
Round 1's two BLOCKING keys and most of its `design` keys have no counterpart in round 2 and are carried as resolved.
Round 1 keys that round 2 still raises under the same area are open: card 14 wiring (F6), card 19 texts (F8), card 4 warning path (F5, F29), card 21 omissions (F11), thin rows (F7), card 6 duplication (F21) and the reader-set split (F25).

The new BLOCKING key is `card20-recoverbatch-test-not-in-edit` (F1).
The new gating `design` keys at MEDIUM are `card21-no-gate-arrival-test-misses-failed-send` (F2) and `card21-ended-shells-logged-where-no-removal-happens` (F3).
The remaining round 2 LOW and NIT findings are not each keyed; the `design` LOWs F12, F17 and F20 are carried above.
