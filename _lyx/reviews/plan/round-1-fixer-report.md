# Plan round 1 — fixer report

## Changes

- F1, fixed as proposed: card 4's Intent says `checklistVisitsOf` also returns every visit in history order, each carrying its `row` field, as a third result the unreadable-recipe list walks.
  Card 5's sentence now adds only the history index, the goto flag and the goto lines to that list.
- F2, fixed as proposed: re-sign arrows on card 4's `checklistVisitsOf` and `stuckEntry` bullets and on card 7's `checklistVisitsOf` bullet, with the heads the finding gives.
- F3, fixed by a placement the gate admits: `caller-uncovered` admits only a re-signed member's own card, so the callers inside `DiscussionSpec` and `PlanSpec` (deleted by card 18) block an arrow on the two directives.
  A deleted member admits earlier cards, so card 19 now deletes `planTwillDirective` and `discussionTwillDirective` and creates `planLensDirective` and `discussionLensDirective`, which take the fan name, with the same bodies.
  `twillLenses` keeps its name and gets an arrow, `func twillLenses(dirs configengine.Dirs, fan, key string) ([]twilldirective.Lens, error)`, since its only callers are the two deleted directives.
  The Intent records why the pair goes and isn't re-signed.
- F4, fixed as proposed: card 11's `TestDiscussionTable_Refuses` and card 12's `TestPlanTable` gain an unparsable-`burler.yaml` row under twill, and card 19's Tests make both rows set the lens key and expect its name.
- F5, fixed as proposed: card 17 lists `internal/shedbuild/doc.go`, and its Intent has the package doc drop the single-LLM constructor.
- F6, fixed as proposed: card 17 names `TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact` and `TestAttach_RunningRecordSatisfiedFileContract_HarvestsNotRespawn` by member glyph, and its Intent bullet names the two tests.

`lyx loom validate-plan` passes with `ok: true`.
Its one informational `caller-uncovered` finding, about `SingleLLMProducer.Call` in `internal/proc/namedjob_windows.go`, comes from a name-only method match, and this round's edits didn't touch that area.

## Disputed

None.

## Deferred

None.
