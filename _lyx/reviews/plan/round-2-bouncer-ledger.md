---
round: 2
ledger:
  - key: status-commit-splits-rework-round
    rounds: [1, 2]
    status: resolved
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
    rounds: [1, 2]
    status: resolved
---

Round 1 raised four findings, and its fixer resolved three of them: `first-card-check-outside-planparser`, `generation-round-rule-three-wordings` and `card10-skip-seam-bound-thin`.
Those three stay resolved in round 2.

`status-commit-splits-rework-round` recurred in round 2.
Round 1's card 16 released the status-commit hold once the highest round's working-tree `record.json` carried a `class`.
Card 5 writes that class before the round commit, so a `Commit` failure or a crash in that window could land a classed record at HEAD.
`committedRejections` would then route PR-Rework to `finish(true)` with no round commit.
The round-2 fixer rekeyed the hold to the pending-rejection record, which is untracked and is cleared only after the round commit.
The finding is now resolved, and card 16's tests cover the classed but uncommitted state.

`verify-omits-planglyph-integration` also recurred in round 2.
It is resolved: `./internal/planglyph/...` is now on both tagged verify lines.

Round 2 raised no new finding, and the plan is approved.
