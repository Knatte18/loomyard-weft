---
round: 2
ledger:
  - key: no-hold-reread-between-check-and-beam-steps
    rounds: [1]
    status: resolved
  - key: fix-task-run-not-moved-under-take-hold
    rounds: [1]
    status: resolved
  - key: parent-warning-left-unreachable
    rounds: [1]
    status: resolved
  - key: card-9-batch-gate-meets-importerless-adoptrecord
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fixhold-clearhold-success-over-refused-replay
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: takeover-retry-drops-parent-records-commits
    rounds: [1]
    status: resolved
  - key: one-subject-ways-forward-under-multi-subject-holds
    rounds: [1]
    status: resolved
  - key: dobby-runs-pattern-summary-line-stale
    rounds: [1]
    status: resolved
  - key: holdwrite-file-header-falsified
    rounds: [1]
    status: resolved
  - key: endsubject-doc-names-adopt-worker
    rounds: [1]
    status: resolved
  - key: card-8-misstates-fronttask-check-run
    rounds: [1]
    status: resolved
  - key: replay-parent-hold-lost-branch-untested
    rounds: [1]
    status: resolved
  - key: parsereplayholds-duplicates-parseowners
    rounds: [1]
    status: resolved
  - key: green-end-removes-fix-task-before-hold-judged
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fixtask-filer-clause-overrides-foreign-takeover
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: green-seam-cannot-carry-rerun
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: here-test-left-on-retired-adoptrecord
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: takeover-archive-retakes-held-teardown-lock
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: takeover-rebase-abort-branch-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card-2-extends-a-doc-it-rewrites
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised thirteen findings; round 2's fresh review no longer raises eleven of those keys, so they are resolved.
Two carry on: the test-only-code guard (now BLOCKING, since it runs at every card gate and cards 2, 3, 5, 7 and 8 leave
functions without a main-reachable caller) and `FixHold`/`ClearHold` reporting success after a replay that dropped
their change.
Round 2 adds seven keys: the green end removing the fix task before the hold is judged, the fix-task filer clause
overriding a foreign takeover entry, the `Green` seam lacking the re-run, the `--here` test left on `adoptrecord`,
`TakeOverRun` re-taking its held teardown lock, the untested rebase-abort branch, and card 2's `Extends` on a doc it rewrites.
