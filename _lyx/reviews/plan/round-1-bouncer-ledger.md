---
round: 1
ledger:
  - key: card3-test-fixture-member-glyph
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card2-reachability-wording-git-rows
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round; there is no earlier ledger.
F1 maps to `card3-test-fixture-member-glyph`: card 3 should name `TestFabricVocabularyFailures_Fixtures` as a member glyph rather than the test file path.
F2 maps to `card2-reachability-wording-git-rows`: card 2's sentence on test-row reachability also covers the two `git diff` rows, which never reach the new reader function.
Neither is gating.
