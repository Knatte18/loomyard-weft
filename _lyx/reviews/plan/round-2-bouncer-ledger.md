---
round: 2
ledger:
  - key: affected-tests-owner-fixtures-and-adopt-asserts
    rounds: [1]
    status: resolved
  - key: judgeorigin-owner-read-no-adopt-subject
    rounds: [1]
    status: resolved
  - key: run-process-owner-background-endrole-drift
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: missing-resign-arrows-three-members
    rounds: [1]
    status: resolved
  - key: drop-local-stop-names-moved-task
    rounds: [1]
    status: resolved
  - key: land-halt-member-needs-board-move
    rounds: [1]
    status: resolved
  - key: beam-await-dobby-down-no-notice
    rounds: [1]
    status: resolved
  - key: beam-cancel-names-refused-landing
    rounds: [1]
    status: resolved
  - key: start-slug-with-all-no-way-forward
    rounds: [1]
    status: resolved
  - key: checkbeampayload-way-forward-task-slug
    rounds: [1]
    status: resolved
  - key: ownedelsewhererefusal-no-caller
    rounds: [1]
    status: resolved
  - key: pawl-nil-seams-test-duplicates-underfilled-env
    rounds: [1]
    status: resolved
  - key: draft-check-skip-no-stated-bound
    rounds: [1]
    status: resolved
  - key: ensurelive-unreachable-refusal-after-card-24
    rounds: [1]
    status: resolved
  - key: ensureparentdobby-no-root-form
    rounds: [1]
    status: resolved
  - key: runprofile-doc-stale-pawl
    rounds: [1]
    status: resolved
  - key: comments-name-removed-verbs
    rounds: [1]
    status: resolved
  - key: testrunownerverdict-on-no-card
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: battencli-resume-claim-tests-unnamed
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: testsetowners-fixture-helpers-unnamed
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: orch-role-stencil-adopt-slug
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: localownerverdict-no-adopt-subject
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised 17 findings; round 2 did not re-raise 16 of them, so they are resolved.
The one round-1 key that returns is the `PATTERN-run-process-owner` line drift, now a `decision` finding: the plan cannot reword it until the parent records the change in the decision record.
Round 2's new findings are fixture and stencil gaps in cards 3 and 25 and the `localOwnerVerdict` adopt subject; each has a local fix the review text states, and none is `design`.
