---
round: 3
ledger:
  - key: fabricengine-snapshot-tests-import-cycle
    rounds: [1]
    status: resolved
  - key: protected-refusal-skips-retry-and-judgment
    rounds: [1]
    status: resolved
  - key: exhausted-push-retried-as-transport
    rounds: [1]
    status: resolved
  - key: unset-run-records-seam-surfaces-mid-push
    rounds: [1]
    status: resolved
  - key: seam-test-search-misses-indirect-readers
    rounds: [1]
    status: resolved
  - key: moved-board-tests-lose-fixtures
    rounds: [1]
    status: resolved
  - key: codetip-test-keeps-shed-imports
    rounds: [1]
    status: resolved
  - key: run-lock-adapter-check-premature-and-duplicate
    rounds: [1]
    status: resolved
  - key: seam-installer-test-cites-unreachable-helper
    rounds: [1]
    status: resolved
  - key: uncontained-write-allowlist-reason-false
    rounds: [1]
    status: resolved
  - key: recorded-run-reads-decode-shed-record-in-fabric
    rounds: [1]
    status: resolved
  - key: outcome-class-doc-listed-under-extends
    rounds: [1]
    status: resolved
  - key: remote-only-needs-tips-before-no-origin-answer
    rounds: [1]
    status: resolved
  - key: ended-teardown-rule-run-and-lock-unnamed
    rounds: [1]
    status: resolved
  - key: in-package-push-tests-meet-seam-refusal
    rounds: [2]
    status: resolved
  - key: reset-protected-refusal-changes-record
    rounds: [2]
    status: resolved
  - key: moved-board-tests-hand-write-shed-record
    rounds: [2]
    status: resolved
  - key: moved-board-tests-duplicate-shared-support
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: push-assertion-kit-asserts-on-code-under-test
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: moved-board-tests-script-foreign-seams
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's keys were resolved by round 2.
Round 2's keys on the seam refusal, the leased reset's protected refusal and the hand-written shed record
are resolved in round 3.

`moved-board-tests-duplicate-shared-support` is open for a second round.
The notifier fake and the fixture types are gone, and the git-using helpers are still restated in `boardengine_test`.

Two keys are new in round 3, both on card 12's test support:
the planned `internal/testkit/pushkit` kit holds assertions on the code under test,
and the scripted teardown and owner seams feed fabric another engine's made-up output.

The recurring theme is the support code of the moved board tests.
