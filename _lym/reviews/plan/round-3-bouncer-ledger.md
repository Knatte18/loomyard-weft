---
round: 3
ledger:
  - key: card8-new-refuses-darn-before-card11
    rounds: [1]
    status: resolved
  - key: description-commit-uncarried
    rounds: [1]
    status: resolved
  - key: batten-nillegal-discussionhistory
    rounds: [1]
    status: resolved
  - key: perchhardcap-ignores-tapestry-yaml
    rounds: [1]
    status: resolved
  - key: card6-missed-moved-key-readers
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card9-stale-docs-preflight-checkseed
    rounds: [1]
    status: resolved
  - key: gate-slot-background-stale-validators
    rounds: [1]
    status: resolved
  - key: not-committed-refusal-darn-write-in-flight
    rounds: [1, 2]
    status: resolved
  - key: card19-beam-refusal-departs-d2-d11
    rounds: [1]
    status: resolved
  - key: card11-reportengine-doc-darn-setup
    rounds: [1]
    status: resolved
  - key: batchercheck-cancel-row-unreachable
    rounds: [1]
    status: resolved
  - key: card5-loomcli-doc-recipe-override
    rounds: [1]
    status: resolved
  - key: card11-wrong-card-cite
    rounds: [1]
    status: resolved
  - key: tapestry-run-wide-key-routing
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: shed-beam-seed-no-reseed-way-forward
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card10-duplicate-drift-verify-edge-test
    rounds: [2]
    status: resolved
  - key: card17-runrecordbullets-missing-resign-arrow
    rounds: [2]
    status: resolved
  - key: card18-duplicate-escalate-judged-first-assertion
    rounds: [2]
    status: resolved
---

Round 3 raised 3 findings, down from 7 and 13.
`card6-missed-moved-key-readers` is open for a third round: the key-literal search was extended, but tests that read a changed value indirectly remain (the `TestMigrateConfig` read-back of `PlanLenses`, and the tests pinning `loomrecipe.EntryNumbers` numbers across cards 9, 10 and 11).
`tapestry-run-wide-key-routing` narrows from a routing defect to one missing named test row in card 6.
`shed-beam-seed-no-reseed-way-forward` narrows to a LOW harness gap: the status row of the refusal-envelope test returns before it asserts the suffix.
The round 2 test-economy, re-sign-arrow and not-committed-refusal keys are resolved by the review's rejected section.
