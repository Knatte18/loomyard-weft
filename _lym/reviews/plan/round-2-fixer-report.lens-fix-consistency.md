```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for agreement with sibling cards, decisions, criteria and cross-references (fix-consistency lens).

1. **LOW**: card 7, the `TestRunWorker` heading "one row per D2 criterion the `TestAdopt` rows do not reach".
   - Summary: the rewritten heading calls every row a D2 criterion, but some of the remaining rows pin D1 criteria.
   - Evidence: the row "An invalid request (`--keep-local` with `--drop-local`) is refused before the lock and any step" and the row "Other subjects are refused before any step" match decision-record D1 criteria. The first is "a direct `lym adopt worker` call ... with invalid arguments is refused before any step"; the second is the other-subjects half of that same criterion. The heading should read "one row per D1 and D2 criterion the `TestAdopt` rows do not reach".

2. **LOW**: card 6, the new F6 sentence "`runBeamSteps`' and `heldBeamWayForward`'s docs, since the landing resumes by hand with `lym tender resume` and `lym loom resume`".
   - Summary: as written, the docs would always name both commands, but card 6's step 6 names `lym tender resume` only for a paused beam.
   - Evidence: step 6 says `heldBeamWayForward` "names `lym tender resume <slug>` and `lym loom resume` in its pair, and otherwise `lym loom resume` in its pair alone, since no entry stands for the tender resume to lift." The doc rewording should be conditional the same way: `lym loom resume`, after `lym tender resume` when adopt paused the beam.

No other twin was left stale. I checked the following against the changes and found them consistent:
- **Seeding without `root`**: card 6 matches `envelopeOf` in card 7.
- **`StoppedPart`**: cards 3, 6 and 7, the overview's Shared Decisions and the decision record's added entry all agree.
- **Reconcile warnings in `stepOwner`**: these agree with the `stepRun` origin report and with card 7's two-process merge row.
- **Dropped `TestRunWorker` rows**: card 7's new `TestAdopt` record assertions and card 6's held-back and failed-deploy rows cover them.
- **Card 1's dropped beamcli target**: no other card references it.
- **Card 8 and card 9 test rewording**: these still meet the D1 and D9 acceptance criteria through `TestFollow` and `TestAdoptSectionOf`.
- **Card 3's lymdirs target, Uses entry and ImpactSummary** all agree.

Further tool calls: none.
