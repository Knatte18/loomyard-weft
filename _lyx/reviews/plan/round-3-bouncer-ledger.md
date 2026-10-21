---
round: 3
ledger:
  - key: new-keys-break-exact-set-pins-and-docs
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: goto-doc-comment-still-says-goto-ends-episode
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card16-guard-refusal-missing-loomcli-doc-go
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: status-help-texts-omit-pause-reason
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card18-comment-in-untargeted-dispositionforhandshake
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: card2-generation-start-skip-no-bound
    rounds: [2]
    status: resolved
  - key: subjectless-bouncer-writes-generation-record
    rounds: [1]
    status: resolved
  - key: rework-record-carry-skipped-on-restep
    rounds: [1]
    status: resolved
  - key: merge-in-patch-id-admission-unbounded
    rounds: [1]
    status: resolved
  - key: case3-reject-comparison-no-settle-time
    rounds: [1]
    status: resolved
  - key: goto-guard-refusal-promises-amendment-for-merge-review
    rounds: [1]
    status: resolved
  - key: batten-budget-clause-placeholder-run-id
    rounds: [1]
    status: resolved
  - key: goto-refusal-way-forward-omits-resume
    rounds: [1]
    status: resolved
  - key: loom-budget-way-forward-placeholder-signature
    rounds: [1]
    status: resolved
---

The forced-edit enumeration gap has now surfaced in all three rounds, each time at a site type the earlier fixes did not name.
Round 3's instance is the reflection-walking `NilSeams` audits: cards 13 and 14 add `Env`, `PRReworkDeps` and `DarnDeps` seams, and `envkit`'s `nilLegal`, battencli's `intentionallyNil` and `FullEnv`'s Darn fill are unlisted.
Round 3 also adds four LOW scope findings on comments and help texts that a card's Intent changes but whose glyph the card does not target.
Round 2's `card2-generation-start-skip-no-bound` was checked and rejected in round 3, so it is resolved.
Round 1's other keys stay resolved.
