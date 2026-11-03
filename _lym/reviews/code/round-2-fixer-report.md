# Fixer-report, code round 2

## Changes

- **F1** fixed as proposed (361fae4c).
  `rewriteCardMentions` in `internal/planparser/renumber.go` skips a `cardMentionRe` span whose preceding byte in `masked` is a hyphen, marking neither a rewrite nor a kept line.
  The `TestRenumber` fixture framing in `internal/planparser/renumber_test.go` gains the paragraph ``Run `lym loom insert-card <first+2> slug` to add one.``; every row compares the overview outside the Card Index, so rows that move card `first+2` fail without the skip.
- **F2** fixed differently (f969a5aa, then reworded after the self-check).
  The proposed sentence "A card id never needs a hand fix: ..." was applied first.
  The `fix-proposal` fork showed it contradicts itself: an id naming the removed card is left as it is, so after `remove-card` it points at a missing card and does need a hand fix.
  `contracts/specs/loom-plan-spec.md:227` now reads "A card id naming a moved card is rewritten and listed under `rewritten_refs`; one naming the removed card is left as it is and is not listed under `prose_mentions`, which holds bare references only."
  This resolves the review's premise (the sentence said a card id is never reported) without the unsound claim.

## Self-check

- fix-proposal, MEDIUM: F2 sentence contradicts itself — fixed with my own change (spec wording above).
- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
