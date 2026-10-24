```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals behind F1, F2 and F3, which the fixer applied as proposed.

**Findings**

1. **LOW** — `internal/shuttleengine/doc.go:426`, which this fix did not touch, now understates F2's behavior.
   - The text is: "Info once it reads live again or no longer reads running."
   - After the F2 change, a run directory whose record has vanished also ends the episode with that Info.
   - A reader of the doc would not expect that third case.
   - Evidence: `activity.go` now calls `waiterlessRuns.Recover` in the `!found` branch.
   - The F2 proposal named only `activity.go`, so this twin was left as it was.

**Proposals that held**

- **F2:** `EpisodeSet.Recover` does nothing for a key with no episode (`internal/logger/episode.go:62-72`). The added `Recover` in the `!found` branch therefore stays silent for run directories that never had a waiterless episode. It cannot double-log, because the key is deleted on recovery. It uses the same key space (`runDir`) as the other two call sites.
- **F3:** The new text, "shuttle: run no longer has a gone waiting process", is accurate on all three recovery paths: live again, terminal outcome and vanished record. The constant is the only source of the string in code (the grep found no other literal), and the test reads the constant.
- **F1:** `requireFailureEpisode` demands exactly one INFO line, so adding stretches before it would have broken it. The fixer put the new stretches after it and counted INFO incrementally (2, then 3) with `countLevelLines`. The counts match the traced episodes: the first recovery is live-again, the second is the terminal outcome, the third is the vanished record.
- **F1, case I tried and ruled out:** the earlier "log names no run-b/run-c" check still reads the stale `log` snapshot taken before the new stretches. That is harmless, because it only checks that other runs are not named.

Calls made:
1. `grep` of the `EpisodeSet` methods in `internal/logger/*.go`: my context lacked `Recover`'s no-op and delete semantics.
2. `grep` for "no longer reads running": to find other copies of the old recovery text and its docs.
