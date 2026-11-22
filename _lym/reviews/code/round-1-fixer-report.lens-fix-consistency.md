```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-round change set for stale siblings and twins left unchanged (lens fix-consistency).

1. **LOW — `internal/shedadapters/roundsnapshot.go:25` and `:36-38` (doc comments, unchanged by the fix).**
   - `FixerRerun` is documented as "nil until the fix producer sets a fixer report aside".
   - `fixerRerunRecord` is documented as keeping "why a round's fixer report was set aside".
   - The fix changed both facts:
     - `archiveReport` now records a halt reason even when no report was archived, so `Report` can be "" and nothing was set aside.
     - `archiveWholeRound` now sets `FixerRerun` back to nil.
   - Both comments should say the record keeps the latest halt reason with the report an earlier call archived, if any, and that retiring the round clears it.

2. **LOW — `_lym/landing/summary.md`, `## Merge danger`.**
   - The blast radius is now `shedadapters`, but the diff also changes `burlerengine`, in `cluster.go` and the `RunOpts.Rerun` and fork-audit behaviour.
   - A wrong audit halts burler rounds, so the radius names less than the diff can break.
   - The fix followed F13's wording but dropped the module the original `burler` value covered.
   - The one-word rule leaves `burler` as the value that spans both packages.

3. **LOW — `internal/shedadapters/burlerfix.go`, `Call` doc (lines ~193-194, unchanged).**
   - It says a rerun is told "the report that session left: one archived before this attempt, or one an earlier call's failure exit archived for the same review".
   - It does not say that a failure exit with no report now records its reason, or that attempt 2 replaces the reason while keeping the earlier report.
   - This is a minor omission; a short added sentence would settle it.

The other siblings I checked agree with the change:
- the summary's push bullets and known gaps;
- the `mutatingGitCommands` doc;
- the way-forward constant against `fixViolationWayForward`;
- `pushFrom` being set on every violation;
- the new test rows against the code paths.

Further tool calls: none; everything I needed was already in context.
