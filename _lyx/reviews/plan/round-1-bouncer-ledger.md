---
round: 1
ledger:
  - key: status-commit-splits-rework-round
    rounds: [1]
    status: open
  - key: first-card-check-outside-planparser
    rounds: [1]
    status: resolved
  - key: generation-round-rule-three-wordings
    rounds: [1]
    status: resolved
  - key: card10-skip-seam-bound-thin
    rounds: [1]
    status: resolved
  - key: verify-omits-planglyph-integration
    rounds: [1]
    status: open
---

Round 1 raised four findings against the plan and the fixer addressed all four.

`first-card-check-outside-planparser`, `generation-round-rule-three-wordings` and `card10-skip-seam-bound-thin` are resolved in the edited plan: card 7 moves the `first_card` comparison into `internal/planparser` with a `planglyph.ValidateRework` entry point and a documented `rework-first-card` ID, the overview states the round rule once with the class condition and names card 9's HEAD reading as authoritative, and card 10's recipe comment carries D7's full bound.

`status-commit-splits-rework-round` stays open.
Card 16 holds the status commit back while the highest round's working-tree `record.json` lacks a `class`, but card 5 writes the class before the round commit.
A commit-seam error or a crash between the two lets a status commit land a classed record at HEAD, after which `committedRejections` routes PR-Rework to `finish(true)` and the round commit never runs.

`verify-omits-planglyph-integration` is new this round: card 7 targets `internal/planglyph`, whose integration-tagged tests the verify section never runs.
