---
round: 1
focus:
  - "Card 6, Edit list: the Intent says `renderFork` (test helper in `internal/websterengine_test`) takes the pack, and the glyph carries no re-sign arrow. Settle whether its parameters change, and check every other Edit glyph whose Intent changes parameters or results (`forkTemplateMarkerValues`, `recoveryTemplateMarkerValues`, `bindCardOne`, `postBatchChecks`) for the same."
  - "Card 7, `buildKickstartPack`: the plan prescribes a transient refusal when the fresh `ParsePlan` fails. Settle whether a named input and a named `TestBeginBatch_KickstartPack` row reach that branch, and whether `ValidateDispatch` upstream already rejects the input."
  - "Card 4, `TestRecordBatch_BindTrace` rows for the `BindHandles` error (read-only plan directory) and the `Delta` plain-error paths: settle whether each named input reaches its branch, or is rejected by an earlier check or by the test environment."
  - "Test economy across cards 2, 4, 6 and 7: the nil-pack and `language: none` rows in card 6's `TestRenderPrompts` and card 7's `TestBeginBatch_KickstartPack`, and the new top-level tests `TestRecordBatch_BindTrace`, `TestKickstartPack`, `TestBeginBatch_KickstartPack`. Settle whether an existing test in the same package already covers a behavior, naming it."
  - "Attack-surface: the `kickstart_pack` switch (cards 5 and 7) lets the fork skip its quarry lookup for resolved entries, and the degraded pack and spans stale within a batch bound that skip. Settle whether the cards' Intents state what is skipped and what bounds it."
  - "Fidelity: walk every Decision D1-D9 and every Constraint in `_lyx/discussion/decision-record.md` to a carrying card, and look for card scope the record does not license (for example the `PersistRecoveryTerminal` change and the `Number` field in card 4, the `Edit` of `planglyph/doc.go` in card 2)."
  - "Card order and sizing: card 3 bundles the `OwnedHandles` move with the outcome types, and card 6 bundles stencils, claims rows, render signature and test helpers. Settle whether any card depends on a later one, and whether either card is more than one independently reviewable unit."
  - "`## verify:` section: settle whether every package a card targets has its tests run, and whether the section's order follows the plan template's order."
---

Seed rationale: entries name where to look. None repeats a check the gate already runs, a missing derived dependency field, or an `ImpactSummary` on a type that does not require one.
