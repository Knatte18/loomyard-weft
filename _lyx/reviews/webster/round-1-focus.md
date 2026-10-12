---
round: 1
focus:
  - "The three pack-on stencils (`webster-fragment-fresh-read-pack.md`, `webster-fragment-card-read-pack.md`, `webster-fragment-locate-pack.md`): do the new texts match the card's quoted lines byte for byte, including the 3-space continuation indents, and do the untouched lines stay byte-identical?"
  - "Pack-off, recovery and verify-fix prompts: does the diff leave them rendering byte-identically, with the nil-pack and recovery rows in `TestRenderPrompts` unedited?"
  - "`contracts/stencils/claims_test.go` `wordingClaims` rows: does each row carry every `wantAll` and `wantNone` the card lists, and do the `why` texts drop the stale step-1 and member-name phrases?"
  - "`TestRenderPrompts` and `TestBeginBatch_KickstartPack`: are the listed row extensions, the `packStepOne` to `packFirstTurn` rename (every use) and the added `inOrder` pins all present, with no new top-level test and no duplicated coverage of an existing pin?"
  - "`internal/websterengine/doc.go` kick-start pack section: does the prose state the first-turn read of every card and span, the Edit-only rule and Decision 4's bound in place of the member-name rule, with one sentence per line and no leftover reference to step 1 reading spans or to the member-name rule?"
---

Seed focus drawn from the single card in `_lyx/plan/01-pack-on-read-all-first.md`; the base diff review still applies in full.
