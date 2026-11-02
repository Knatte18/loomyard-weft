# Plan round 2 — fixer-report

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  In card 10 (formerly card 9), `endGateFixers`'s slug read error is now returned wrapping the read error, and neither error carries a shed transient mark.
  The `TestEndGateFixers` row now reads "a slug read error is returned and calls nothing".
- **F2 (LOW), fixed as proposed.**
  Card 3 lists `internal/websterengine_test#testReplyPath` under **Edit:**.
  Its comment bullet now covers the `testReplyPath` doc comment in place of the old template_test.go line bullet, and says the three file headers belong to card 4.
  The render.go, strand.go and recoverbatch.go header bullets moved to a new `Prosa` card 4, `webster-file-headers`, added with `lym loom insert-card 4`.
  Card 4's targets are the three file self glyphs, and its Intent states each header's post-change wording.
  Its `Uses` of `internal/websterengine#RecoverSpawnOrAttach`, a card 3 target, orders it after card 3.
  The insert moved old cards 4–13 to 5–14.
  Every bare card number that `insert-card` reported under `prose_mentions` was rewritten by hand, in cards 1, 3, 6, 7, 8, 9, 11, 12 and 13.
- **F3 (LOW), fixed as proposed.**
  Card 10's opening now reads "once shed's own checks and the told guard have admitted the target, and before the move is written".
  The `Relaunch` bullet names both cases that end fixers without a write, an `AdmitRelaunch` refusal and an in-effect consume, and says why the in-effect case ends none that are live.
  `internal/shedengine#GotoGuard` is under **Edit:** and `internal/shedengine/doc.go` is under **Uses:**.
  A new Intent bullet gives the sentence that `GotoGuard`'s doc comment gains.
- **F4 (LOW), fixed as proposed.**
  `internal/loomcli/doc.go` moved from **Uses:** to a new **Extends:** group placed before **Uses:**.
  A new Intent bullet gives the sentence added after the two-arm guard paragraph.
  The card 10 ImpactSummary now notes the two doc sentences.
- `lym loom validate-plan` returned `ok: true` with informational findings only.
- **Self-check fix:** card 3's `testReplyPath` bullet now names both cold fixer prompts the constant serves.

## Self-check

- fix-consistency: no findings.
- fix-claims, LOW: card 3's `testReplyPath` bullet named only the verify fixer's render, but `RenderPlanFixPrompt`'s test passes the constant too. Fixed as proposed, merged with the fix-proposal duplicate.
- fix-proposal, LOW: the same `testReplyPath` finding. Duplicate of the fix-claims finding, fixed with it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
