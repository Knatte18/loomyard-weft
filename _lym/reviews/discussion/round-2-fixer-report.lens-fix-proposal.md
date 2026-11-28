```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F19 as proposed; F20 was changed by the fixer and isn't covered by this lens).

**Proposals checked**
- F19: the held path, the `landed` bullet that now takes the held path, and the reworded D8 criterion.
- F20: the acceptance criterion and the Constraints extension it proposed, both carried out as proposed.

**Findings**

1. MEDIUM. D8 held path: "Its own landing commit is its entry's `landed` commit, or, with no such outcome, the commit on the parent whose `Lym-Landing` trailer names its slug."
   Summary: the held test runs "whatever its entry says", so a run can take the held path with a `landed` commit the parent's head no longer holds, and that commit then becomes its ledger tip and its dobby record.
   Case: an operator resets the parent past c1 (the case D8's second bullet names), then lands the same change again by hand. `HoldsWork` judges by tree equality alone (`internal/fabricengine/parenttips.go:222-246`), so the run reads as held and takes the held path before any outcome bullet. It then records c1 through `Deps.LandLocally` and `Deps.RecordLanding`, but c1 is no longer on the parent. That pushes a ledger tip off the parent's first-parent chain and writes a wrong landed-commit record.
   Evidence: the `landed`-not-on-parent bullet that clears the outcome is reached only "Otherwise", after the held test.
   Change: take the entry's `landed` commit only while the parent's head holds it, and fall back to the trailer commit otherwise.

2. MEDIUM. The same sentence, "the commit on the parent whose `Lym-Landing` trailer names its slug".
   Summary: the sentence gives no search range and no rule for which commit wins, and the only existing reader cannot answer it once the landing has been pushed.
   Evidence: `landingCommitsOn` and `LandingCommits` read only commits past the tracking ref, and "A run named twice keeps its newest commit" (`internal/fabricengine/landing.go:111-126, 128-131`).
   Case: a run with no entry whose landing origin already holds finds no trailer commit through that reader. It falls to "the parent's head". When other runs have landed since, that head names another run's commit in the dobby record, the wrong-commit defect F20 raised.
   A run that landed twice (D8 criterion "task branch gained commits after it was marked `landed`") has two trailer commits, and the text doesn't say to take the newest.
   Change: name the range (the parent's first-parent history, not only the commits past upstream) and say the newest commit wins.

Tool calls:
- grep of `LandingTrailerKey` readers: my context held no code that reads the trailer back.
- Read of `internal/fabricengine/landing.go:95-134`: to see which range that reader covers and which commit wins when a run is named twice.
