```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F14 proposal the fixer carried out in card 3 (fix-proposal lens).

no findings

I tried these cases against the proposal. None makes the artifact wrong where it was right:
- **Run first seen past Land.** This covers a fresh record, a record from an older binary without `level`, and a level that failed through every tick ahead of Land. The entry holds no `Level`, so step 2 reads past Land and gets the inherited value. That is the same limit the landing notice accepts. The in-memory `watchLevels` design was no better: it was empty after a restart. Not made worse.
- **Operator changes `run_watch` ahead of Land.** It is still honoured, because step 2 reads on every tick while the run is not `PastLand`.
- **Card 6 creating an entry "whatever the run's watch level".** An entry with no `Level` is treated as unread, which agrees with the new step 2 and with the clear hold ("no `Watches` entry or whose entry holds no `Level`").
- **`TestClearDoneRuns` second row.** The run is done and its entry holds `Level` `stops`, so step 2 does not call `Level` and the owed line posts. The row's outcome is unchanged.
- **New `TestRunWatch` rows.** Neither row sets land_notice, so the copy is off or never read and the landed reading is never skipped. Their expected lines (none under a stored `off`, one under a stored `stops`) follow from the changed step 2 and step 4.
- **Upkeep drops an entry, then the run is listed again past Land.** The drop needs the run unlisted, `StatusExists` false and nothing owed. Under the old design the in-memory `watchLevels` survived that drop; now `Level` goes with the entry, so a relisted run would read the inherited value. That needs a batten status to vanish and come back, which no flow in the artifact or the cited code produces, so I don't raise it.

No further tool calls made.
