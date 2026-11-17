---
round: 3
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
    status: resolved
  - key: fixhold-clearhold-success-over-refused-replay
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: fixtask-filer-clause-overrides-foreign-takeover
    rounds: [2]
    status: resolved
  - key: green-seam-cannot-carry-rerun
    rounds: [2]
    status: resolved
  - key: here-test-left-on-retired-adoptrecord
    rounds: [2]
    status: resolved
  - key: takeover-archive-retakes-held-teardown-lock
    rounds: [2]
    status: resolved
  - key: takeover-rebase-abort-branch-untested
    rounds: [2]
    status: resolved
  - key: card-2-extends-a-doc-it-rewrites
    rounds: [2]
    status: resolved
  - key: takeover-teardown-lock-refusal-unreachable
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: takeover-rebase-abort-tip-ambiguous
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gained-deps-fields-have-no-named-setter
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hold-seam-cannot-supply-machine-name
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: greendeps-dir-becomes-dead
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised thirteen findings and round 2 nine; round 3 raises five, none BLOCKING.
Every key open after round 2 is gone from round 3's review, so all of them are resolved here.
The two long-running keys, the test-only-code guard against card 9 and the `FixHold`/`ClearHold` success over a refused replay, no longer appear.

Round 3's one MEDIUM design finding is new: `TakeOverRun`'s two teardown locks refuse with no input reaching the refusal, and the wait cannot be shortened.
Its fix adds a `LockWait` field and a test row, so it is marked `design-wrong` and needs a fresh review.
The four LOW findings are local corrections to cards 3 and 8 whose text fixes the result.
