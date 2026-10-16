# Plan round 1 — fixer report

## Changes

- F1, fixed as proposed: card 9's `renderRecoveryWithUncommitted` and `renderMaster` lines and card 13's `renderForkWithPack` and `renderRecoveryWithUncommitted` lines now carry re-sign arrows.
  Each arrow is the helper's current signature in `internal/websterengine/template_test.go` plus the parameters its card's Intent adds.
- F2, fixed as proposed: card 8's Intent states D5.4's bound after its opening paragraph, naming cards 5, 7 and 17.
  The `pattern-fragment-changes.md` bullet adds that only the entries listed there are judged by their listed text.
- F3, fixed as proposed, folding the Bouncer case into the Burler-round row: card 10's refusal-spec bullet now plans one `## shed` row for the Burler-round row's review as the correctness halt.
  That row's trigger says the Bouncer's judge call degrades to Stuck, the perch's `on_stuck` routes it to the Burler-round row, and `Engine.Review` there refuses and halts the run.
  This matches `degrade` in `internal/shedadapters/bouncer.go` and the `on_stuck` keys in `contracts/recipes/loom-recipe.yaml`.
- F4, fixed as proposed: card 11 lists `internal/loomcli#loomCLI.wireLightweight` after `validatePlanCmd`, and an Intent bullet updates its doc comment for `validate-plan`'s own `language.yaml` load.
- F5, fixed as proposed: card 1 gains a `**Prosa:**` group holding `docs/overview.md`, and an Intent bullet adds the rename tools to the `internal/languageconfig/` module-tree line.

`lyx loom validate-plan` reports `ok: true` after the edits, with only the informational `create-new-unit` finding on card 8.

## Disputed

None.

## Deferred

None.
