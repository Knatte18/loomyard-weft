---
round: 2
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
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: reset-protected-refusal-changes-record
    rounds: [2]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: moved-board-tests-hand-write-shed-record
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: moved-board-tests-duplicate-shared-support
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's keys are all resolved by the round-2 review's account of them.
Four keys are new in round 2, all BLOCKING, and each follows from a round-1 fix:

- The seam refusal reaches two in-package push tests, and card 3's account of its search result is wrong.
- Card 7's protected-outcome mapping also takes in the leased reset's refusal, a different path from round 1's
  rejected-push refusal.
- Card 12's named fixtures for the moved board tests include a hand-written shed record,
  and restated helpers and `hubforge` wrapper types.

The recurring themes are card 7's outcome mapping against today's behavior, the reach of the required seam,
and the support code of the moved board tests.
