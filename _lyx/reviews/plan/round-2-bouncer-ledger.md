---
round: 2
ledger:
  - key: webster-callsites-edited-three-times
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: profiles-not-declared-open-map
    rounds: [1]
    status: resolved
  - key: tokencount-attribution-ignores-real-transcript-shape
    rounds: [1]
    status: resolved
  - key: card6-repeats-card2-coefficient-error-rows
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: card10-misdescribes-worktree-config-resolution
    rounds: [1]
    status: resolved
  - key: card3-basesha-extraction-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
---

Round 1 opened five findings; round 2 confirmed three were fixed (open-map declaration, tokencount fork attribution, card 10 config resolution).

Two keys persist in new form.
`webster-callsites-edited-three-times`: after the earlier fix, card 4 and card 7 still both edit the batch-forming line in the webstercli verbs and `Run`, with card 8 touching `rebaseline.go` a third time; the proposed fix folds card 4 into the recorded-partition card.
`card6-repeats-card2-coefficient-error-rows`: `TestActive` still carries a negative-coefficient row that duplicates `ProfileWeights` rows and the `verbs_test.go` rebaseline step.

New in round 2: `card3-basesha-extraction-untested`, where the `RunTally.BaseSHA` extraction in `tools/tokencount` has no test and no stated envelope-recognition rule.
