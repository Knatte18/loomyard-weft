---
round: 1
focus:
  - "Card 1 `Rename` of `refMember` to `RefMember` plus a `Create` of `TestRefMember` and an `Edit` of `RefCovers`: settle whether the card's grouping and typing are right, and whether `TestRefMember` rows duplicate what `TestRefFile` or `TestRefCovers` already cover (test economy)."
  - "Card 2 `TestScopeGuard_SymbolCoverage` and its many rows: settle whether each new row reaches distinct behavior or repeats another row or an existing `ScopeGuard` test (test economy), and whether the three `Delete` targets are covered by rows."
  - "Card 2 `Edit` targets `internal/planglyph#ScopeGuard` and `targetGlyphUnion`: settle whether the `Intent` changes a signature, so the re-sign arrow is needed or absent correctly."
  - "Card 2 `symbolFilesCovered` empty-`files` and no-scope branches, and card 3's deviation rule: settle whether each prescribed refusal or guard branch has a named input that reaches it and a named test, and whether an upstream check rejects that input."
  - "Card 3 widens what the fork may change outside targets (three sanctioned deviations): settle whether the `Intent` states what each can let through and what bounds it (attack-surface question)."
  - "Card order and fidelity: settle whether card 2 depending on card 1's `RefMember` and card 3 and 4 pointing at the guard are in dependency order, and whether every D1 to D3 decision and every Constraint in the decision record is carried by some card with no extra scope."
  - "`## verify:` section: settle whether it runs the tests of every package the cards target (`internal/planparser`, `internal/planglyph`, the stencil and spec consumers, `cmd/lyx` for the test-budget file) and is in the template's order."
---

Focus is derived from the plan files and the decision record only.
