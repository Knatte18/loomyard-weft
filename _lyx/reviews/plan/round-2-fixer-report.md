# Plan Burler round 2 — fixer report

Every finding's premise was checked against the tree and held.
`lyx loom validate-plan` passes after the fixes, with only the informational `create-new-unit` finding for the PATTERN file on the scan card.

## Changes

- F1, fixed as proposed: card 8 lists `internal/shedrecipe#websterEntry`, `#zeroEnvField`, `#simpleEntryCases`, `#TestWebsterEntry_SeamFields` and `internal/battencli#intentionallyNil`, and its Intent says `websterEntry` validates `WebsterDeps.Engines` and the test lists and `intentionallyNil` key move to it.
- F2, fixed as proposed: card 10 lists `internal/testkit/envkit#nilLegal`, and its Intent adds `WebsterDeps.InstructionFiles` to `nilLegal`, keeping `NilSeams(FullEnv)` empty and leaving battencli's `intentionallyNil` unchanged.
- F3, fixed as proposed: card 10 lists `internal/webstercli#websterCLI.runDeps`, `#TestWire_StandaloneMode` and `#TestWire_ModeHubSelectsHubMode`; its Intent has `runDeps` copy the lister into `RunDeps.InstructionFiles`, and both wiring tests assert it is non-nil.
- F4, fixed as proposed: the Start refusal's generic clause now reads `give the role a model alias of an engine that meets them, or for a role that takes no alias set engine: in shuttle.yaml to one that does, then re-run the verb`, and the Intent names `lyx orch start` (`orchSpec`) as the caller it serves; the `TestRunner_Requires` unwrapped assertion checks both halves.
  The text now lives in the new card 11 (F8).
- F5, fixed as proposed: card 10's `master engine not registered` row names `validate` and `await-batch` with no state, `validate --batches` and `rebaseline`, and says `run` is refused earlier by the `role engine without fork audit` row.
  The Intent says `executionBatches` and `rebaselineCmd` return the lister's error without the `transient` clause, and the `TestValidateCmd_BatchesFlag` and `TestRebaselineCmd_Refusals` rows assert the `set master in webster.yaml` way forward and no `transient`.
- F6, fixed as proposed: card 10's Intent rewords the existing `Merriam start base unreadable` row to a present instruction file Master's engine names.
- F7, fixed as proposed: card 8's Intent states the engine-less read's let-through and its bound after the `sessionAuditEngine` bullets.
- F8, fixed as proposed: `lyx loom insert-card 11 start-requirements-model-key` added card 11, which holds the `ModelKey`, `ModelKey.Wrap` and `startRequirementsError` Create targets, the `checkRequirementsAtStart`, `shuttleengine/doc.go` and `TestRunner_Requires` Edit targets, the four-form way-forward Intent and its `TestRunner_Requires` test bullet.
  The seat card, now card 12, drops those targets and paragraphs, refers to card 11's `ModelKey`, and its ImpactSummary drops the shuttle clause.
  Card 1's `card 13's scan` mention, the one prose mention insert-card reported, now reads `card 14's scan`; card 14's `cards 1 to 10` stays valid.
  `lens-review.md` keeps its card numbers, since it records round 1 as it ran.

## Disputed

None.

## Deferred

None.
