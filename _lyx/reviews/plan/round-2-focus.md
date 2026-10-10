---
round: 2
focus:
  - "Card order for the discussion stencil: in the plan's cards, find where `{{template \"seat-directive-chair\"}}` is added to `contracts/stencils/loom/loom-template-discussion.md` and where `loomengine.composePrompt` (which fills that stencil through `stencil.FillOptional` with no includes) is deleted or replaced; settle whether any card between them leaves a test or the live Discussion-Write row failing to compose."
  - "Cross-package testkit change: find the card that changes `shedfake.SeatRunner` (`internal/testkit/shedfake`) and settle whether it is sized and ordered as its own reviewable unit, and whether it targets every existing consumer test whose gated table its new behaviour changes."
  - "Double gate evaluation: where a card gives `SeatRunner` a `RunFn` that already evaluates `table.Gate` through a script, settle whether the planned fake also evaluates the gate and whether the card states which evaluator wins."
  - "Decision-record fidelity: compare each Shared Decision in `_lyx/plan/00-overview.md` with `_lyx/discussion/decision-record.md` (D4 and its acceptance criteria on absent key and load-time resolution) and settle whether any plan outcome departs from the record without a recorded decision."
---

Focus targets the fixes most likely to have shifted ordering and scope in the plan after round 1.
