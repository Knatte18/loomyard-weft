---
round: 2
focus:
  - "Card 8's test list: check that the failing-verify test names a policy finding (a fork fabric-reference) on an OK report on the same call, so it exercises step 7's first trigger and asserts the archive, ErrBatchFailed and recover-batch, and that card 11's recovery test refers to that case."
  - "Cross-card consistency after the round-1 fixer edits: check that the renumbered cards 26–31 and every in-card cross-reference (card numbers, 'the same finding'-style antecedents, Applies-to ranges) still point at what they mean."
---

Round 1's fixes all landed; the second entry exists because the one new issue came from an antecedent the fixer's insertions shifted, and the same edits touched several other cards.
