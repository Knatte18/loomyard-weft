```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: checked the `activity.go` changes (the `!found` recovery and the reworded `waiterlessRecovered`) and the new test stretches.

No findings.

- **`!found` branch:** a run directory with an open waiterless episode loses its `run.json`. `loadRunState` returns found=false, so the new `waiterlessRuns.Recover(runDir, ...)` ends the episode with one Info and deletes the key. `EpisodeSet.Recover` (`internal/logger/episode.go:63-72`) does nothing for a key with no episode, so directories that were never waiterless log nothing.
- **Message change:** the constant is used at the three `Recover` sites and in the test, which reads the constant. No literal copy of the old text is left behind. The new text matches none of the Warn or unreadable-record messages, so `countLevelLines` for the Info text counts only waiterless recoveries.
- **Terminal stretch:** the sequence is waiterless then read (a new episode, Warn #2), then done and read, which ends the episode through `noteWaiterlessRun` (Info #2). The extra read is a no-op, so the count is 2.
- **Vanish stretch:** waiterless then read (Warn #3), then remove `run.json` and read, which ends the episode through the `!found` path (Info #3). The extra read is a no-op, so the count is 3.
- **Defects reintroduced:**
  - With the Recover call at `activity.go:124` removed, the Info count stays at 1 and the `want 2` check fails.
  - With the new `!found` Recover removed, the count stays at 2 and the `want 3` check fails.
  - Both new assertions therefore fail with the defect in place.
- **Isolation:** each test uses its own `t.TempDir()` root, so run-directory keys cannot collide with other tests through the package-level `waiterlessRuns`.
- **Stale `log` snapshot (not a defect):** the later run-b/run-c "log names" check reads the `log` snapshot taken before the new stretches. It still covers what it did before, and the new stretches touch only run-a.

Further tool calls: 1. I read `internal/logger/episode.go` (the `EpisodeSet` Fail/Recover semantics), because my context did not hold it and the Recover no-op and re-Fail behaviour decide whether the counts hold.
