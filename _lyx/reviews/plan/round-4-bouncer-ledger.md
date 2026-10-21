---
round: 4
ledger:
  - key: card12-start-generation-name-collision
    rounds: [4]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: rewritten-sentence-carrier-not-listed-glyph
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: new-keys-break-exact-set-pins-and-docs
    rounds: [1, 2, 3, 4]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: nillegal-hides-missing-reject-fill-at-wiring
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: goto-doc-comment-still-says-goto-ends-episode
    rounds: [3]
    status: resolved
  - key: card16-guard-refusal-missing-loomcli-doc-go
    rounds: [3]
    status: resolved
  - key: status-help-texts-omit-pause-reason
    rounds: [3]
    status: resolved
  - key: card18-comment-in-untargeted-dispositionforhandshake
    rounds: [3]
    status: resolved
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

Round 4 opens with a BLOCKING name collision: card 12 creates `Bouncer.startGeneration`, which done card 11 already landed (F1).
F2 is a design finding on the method of choosing the glyph that carries a rewritten sentence, a new site type after round 3's `NilSeams` audits; those audits were checked and rejected this round.
F3 is the enumerating-doc half of the recurring key, now in its fourth round, for a recipe-row key in the shed recipe spec.
F4 is a wiring-test gap that `nilLegal` hides for the reject-file seams.
The four round-3 LOW scope keys do not recur and are resolved, as are round 1 and 2's earlier keys.
