---
round: 1
ledger:
  - key: affected-tests-owner-fixtures-and-adopt-asserts
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: judgeorigin-owner-read-no-adopt-subject
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: run-process-owner-background-endrole-drift
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: missing-resign-arrows-three-members
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: drop-local-stop-names-moved-task
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: land-halt-member-needs-board-move
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: beam-await-dobby-down-no-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: beam-cancel-names-refused-landing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: start-slug-with-all-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: checkbeampayload-way-forward-task-slug
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: ownedelsewhererefusal-no-caller
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pawl-nil-seams-test-duplicates-underfilled-env
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: draft-check-skip-no-stated-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ensurelive-unreachable-refusal-after-card-24
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: ensureparentdobby-no-root-form
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: runprofile-doc-stale-pawl
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: comments-name-removed-verbs
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 17 findings, none BLOCKING; one gates (design, MEDIUM).
The gating finding is a method gap: the fixture sweep for cards 3 and 25 found tests built through shared owner helpers, but not tests that seed a task `owner_machine` or assert a slug-named adopt directly.
The two proposed design-wrong fixes add a new `Deps` seam (`DobbyLive`) and drop a refusal and its test row, so a fresh review should read that new text.
Every other fix is a local correction the finding text already fixes.
