---
round: 1
focus:
  - "Card 2 (`02-pack-per-card.md`): its Edit group lists many members whose `Intent` may change parameters, results or receivers (`BeginBatch`, `RenderRecoveryPrompt`, `renderCardPointers`, `Index.KickstartPack`, `Pack`); settle whether each such member carries the `->` re-sign arrow, and whether the card is one reviewable unit or should split."
  - "Cards 2 and 3 both edit `RenderForkPrompt`, `RenderRecoveryPrompt`, `contracts/stencils#entries`, `wordingClaims`, `TestRenderPrompts` and `TestBeginBatch_KickstartPack`; settle that card 3 does not depend on a symbol or fragment card 2 has not yet created (card order), and that card 2's `renderLocate` signature and card 3's `webster-fragment-locate-pack.md` edit are consistent."
  - "Decision record fidelity: settle that every Decision (D1-D6) and every Constraint in `_lyx/discussion/decision-record.md` is carried by some card, in particular the byte-identical pack-off, glyph-opt-out and recovery-prompt constraints, and that no card adds scope the record does not license."
  - "Test economy: card 1 adds fixture rows to `TestKickstartPack`, and card 2 adds a `oneBatch` case field and new rows; settle whether any planned new test or case duplicates behavior an existing test already covers, naming that test."
  - "Attack-surface and reachability: for the new pack-on/pack-off branching and any refusal or guard branch a card prescribes (unresolved-pack fragment, error returns of `kickstartPack` and `renderCardPack`), settle that a named input and a named test reach it and that its `Intent` states what it can skip and what bounds it."
  - "Verify coverage: settle that every package a card targets (including `contracts/stencils` and `internal/websterengine_test`) has its tests run by `## verify:`, and that the section order matches the plan template."
---

Seed focus for a three-card plan; each entry names a place to look and a question, none pre-states a verdict.
