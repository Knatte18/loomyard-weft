```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as proposed (F1–F8) and report only cases where the proposal itself is wrong.

**Findings**

1. MEDIUM, F3. Changed text: `finalizeEnteredRead`'s `how == endedDone && status.CurrentProducer == raddleFileRow` in `internal/fabricengine/replay.go`.
   - **Summary:** The exemption now covers only `endedDone`, but `readEndedSet` also reports `endedRemoved` for a done entry origin later pruned.
   - **Evidence:**
     - `replay.go:139-158` marks any run absent from the tip but present in the ledger history as `endedRemoved`, with no check that it was done first.
     - The review itself says the pair status stays at Raddle-File after the run is done, until batten's Land or a later status takes over.
     - Raddle-File ends the entry done, so a `board prune` of done entries on origin during that window turns the entry from done into pruned.
     - The reconcile then reads it as `removed`, which is no longer exempt.
     - The pair is torn down, or its commits are set aside, while batten still has Land and Worktree-Teardown to carry out. That is the defect the original change fixed.
   - **Untested:** The new test covers only an entry deleted from the board outright. Nothing covers a done-then-pruned entry.

2. LOW, F1. Changed text: `claimStart` calls `c.guard.ClearMotifRun`, which is `ClearMotifRun(location, slug)` in `hubStartGuard`.
   - **Summary:** The board is no longer re-read at removal time, though `refuseStartOwnedElsewhere` pulls the board in between.
   - **Evidence:**
     - The seam `ClearsMotifRun` judged the entry raddle before the pull.
     - `claimStart` then runs `refuseStartOwnedElsewhere`, which calls `c.owner.Pull()`.
     - `ClearMotifRun` itself checks only that the run is done and its child recipe is motif (`cleanup.go:147-156`).
     - If the pull brings an entry that was raddle back to darn, or removes it, the motif run is still removed.
     - The old closure re-read the board at clear time, so this is a narrowed guarantee.

3. LOW, F8. Changed text: the `sketchRefusedNotice` payload `{"slug":…,"kind":"note","labels":["sketch"],"status":null,"recipe":"","depends_on":[]}`.
   - **Summary:** The payload is not equivalent to `MakeSketch`, and no test runs it against the board.
   - **Evidence:**
     - `sketchOf` (`sketch.go:44-62`) keeps the entry's non-type labels. The payload replaces all labels with `["sketch"]`, so a label such as `board` is lost.
     - `sketchOf` also clears `profile`, `run_settings`, `pull_request`, `isolated` and `notify_on_land`. The payload leaves them set.
     - The test (`commitstatus_integration_test.go`) only asserts the string. It never applies the upsert, so whether the board accepts it is unverified.

4. LOW, F4. Changed text: `startClearsMotifRun` now calls `boardengine.OpenHub(location.HubPath)` for every released task with a done run record.
   - **Summary:** `DobbyTasks` now loads board config from disk per tick for tasks it used to skip cheaply.
   - **Evidence:**
     - The old `task.Recipe != raddle` check returned before any I/O.
     - A done record under a darn task persists until cleaned up, as `dobbyengine/doc.go:111` says.
     - Each such record now costs a config load on every tick.
     - A board that fails to open logs a plain `logger.Warn` on every tick. The file already uses `logger.EpisodeSet` for this kind of repeating failure.

F2, F5, F6 and F7: no findings.

Further tool calls: none. My context already held every file the lens needed.
