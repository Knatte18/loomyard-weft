```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if the behaviour broke.

**Per behaviour, the test that pins it**

1. **A write giving a new entry the arras recipe is refused and writes nothing.** Pinned by `TestUpsertRefusesTheArrasRecipe/ANewEntry` (`internal/boardengine/store_test.go:398`, assertions at :411-418).
2. **A write changing another recipe to arras is refused, and the prior recipe stays.** Pinned by `TestUpsertRefusesTheArrasRecipe/AnEntryOfAnotherRecipe` (`store_test.go:399`).
3. **The refusal names `"recipe":"motif"` as the way forward, and that write is taken.** Pinned by `TestUpsertRefusesTheArrasRecipe` (`store_test.go:412`, :420-425).
4. **An entry that already holds arras keeps taking writes.** Pinned indirectly by `startArrasRun` (`internal/boardengine/raddlefile_test.go:321-332`). It sets arras in the store and then calls `SetStatus`, which passes through `validateWrite`. If the exemption broke, the Fatalf would fail every arras family-filing test.
5. **The other write paths (batch, merge, promote, set-deps, replay, raddle repoint, MakeSketch) refuse arras.** They share `validateWrite` (`store.go:478`, :484-486), so the UpsertTask rows cover the shared check. No per-path row is needed.
6. **`RunView.Motif` is read from the child-recipe record.** Pinned by `TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitTheEndedEntryThePairAndTheMotifRecipe`: "landed" has motif and reads true, "unlanded" has tapestry and reads false, and runs with no record read false (`internal/beamcli/dobbypairanchor_test.go:247-251`, :277-278).
7. **A failed child-recipe read.** No test. The new read returns its error into the run's read set (`internal/battencli/dobbyreads.go:230-234`). No row plants an unreadable child-recipe record, as the "unreadable" row (`dobbypairanchor_test.go:295`) does for the landed-commit file.
8. **A landed motif run sends the Sketch line with `land_notice` off.** Pinned by `TestLandNotices/ALandedMotifRunTellsOfItsSketchWithTheNoticeOff` (`internal/dobbyengine/landnotice_test.go:523`).
9. **With the notice on, the Sketch line replaces the landed line.** Pinned by `ALandedMotifRunSendsOnlyTheSketchLineWithTheNoticeOn` (:524).
10. **The Sketch line goes even when `land_notice` never read.** Pinned by `ALandedMotifRunNeverReadAheadOfLandTellsOfItsSketch` (:525).
11. **A motif run that ends without landing follows the setting.** Pinned by `AMotifRunEndingWithoutLandingFollowsTheSetting` (:526). It covers only the off reading. The on reading reaches the unchanged `landNoticeLine` branch, which the existing unlanded rows cover.
12. **A motif run first seen past Land sends nothing.** Pinned by `AMotifRunFirstSeenPastLandSendsNothing` (:527).
13. **The Sketch line's exact text.** Pinned by `sketchLine` (`landnotice_test.go:515`), compared by equality in rows 8-10.
14. **The `lym board upsert --help` recipe list, the orch role, role-autonomy, the skills, the raddle stencil, the rubric and the doc comments.** These are static prose and exempt. No new prose assertion was added.
15. **The `startArrasRun` change.** It sets up the arras recipe differently because no write can now give it. It loosens no assertion.

**Findings**

- **LOW · design** · `internal/battencli/dobbyreads.go:230-234`: the failure path of the new child-recipe read has no test row.
  - What it does: the read's error goes back through the run's read set, with no Warn-and-continue as the landing-commit read has.
  - What would catch a break: a row in `TestDobbyRuns_ReadsTheLandOutcome…TheMotifRecipe` that plants a directory where the child-recipe file belongs, as the "unreadable" row does at `internal/beamcli/dobbypairanchor_test.go:295`. It would pin what a failed read does to the run's view and to the rest of the listing.
  - Evidence: `dobbyreads.go:230-234` and `dobbypairanchor_test.go:247-251`, which only write readable records.

Tool calls:
1. Read `internal/battencli/dobbyreads.go:195-260` and grepped the "unreadable" rows of `dobbypairanchor_test.go`. My context had only the diff hunk of the read set, not how its errors are handled or which failure rows exist.
