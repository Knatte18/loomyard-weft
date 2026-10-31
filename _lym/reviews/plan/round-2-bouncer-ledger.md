---
round: 2
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
    rounds: [1, 2]
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
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
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
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: shed-beam-seed-no-reseed-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: card10-duplicate-drift-verify-edge-test
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card17-runrecordbullets-missing-resign-arrow
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card18-duplicate-escalate-judged-first-assertion
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised 13 findings; round 2 raised 7.
Every round 1 key but two is resolved: the review's rejected section confirms the card 8 ordering, the perch's read of `tapestry.yaml` and the other round 1 fixes hold.
`card6-missed-moved-key-readers` stays open and widens: the affected-set for the key removals and the tapestry move is hand-built, and card 5 itself leaves `internal/loomengine` tests uncompilable.
`not-committed-refusal-darn-write-in-flight` stays open as a decision finding; the card text is accepted as the working one, and the gap is the missing `Added after Discussion` entry that only the parent records.
Two BLOCKING findings are new: run-wide loom keys routed through the tapestry spec (design), and the `lym shed` beam-seed refusal without a re-seed way forward (consistency).
Three LOW findings (card 10, 17, 18) are local test-economy and re-sign-arrow corrections.
