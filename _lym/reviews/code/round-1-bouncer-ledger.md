---
round: 1
ledger:
  - key: watch-loop-no-one-second-floor
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: page-busy-loops-on-absent-board-json
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: board-html-not-pinned-to-lf
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: verb-wiring-behaviors-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unauthenticated-api-request-resets-idle-clock
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: create-request-may-carry-revision
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: create-of-project-slug-answers-as-removed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sync-and-owner-set-commit-contents-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-beam-anchor-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: revision-test-computes-expectation-with-code-under-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: viewer-page-test-needs-keep-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: keybindings-block-test-needs-keep-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings, all open.
The gating ones are the missing one-second floor on watch wakes, the busy loop on an absent `board.json`,
the unpinned `board.html` line endings, and the untested verb wiring.
The rest are LOW: idle-clock stamping before the token check, a revision on create, a project slug answering as removed,
two untested branches, one tautological expectation, and two `//testtiming:keep` marks.
