```yaml
lens: fix-claims
outcome: reported
```

I checked the claims in the fix's added or rewritten sentences against HEAD.

no findings

- The fix says the run status keeps no Stuck reason. That holds: `HistoryEntry.Reason` is only the entry gate's text on a skipped entry (`internal/shedengine/status.go:118`), and the step-3b resume write persists an empty error before the producer runs (`internal/shedengine/run.go:296`).
- The fix's history test reads `Producer`, `Outcome`, `At` and `LastAt`, and the struct has all four. `LastAt` is the time of the most recent folded call, and `At` keeps the first call's time, so "`LastAt` when set, `At` otherwise" is a sound latest time.
- The test assumes a resume leaves a Stuck as the last history entry. It does: `lyx loom resume` appends no history entry, and `RecordResumer` sets only `PendingBy` (`internal/shedengine/goto.go:271`). Only a goto appends an entry (`goto.go:135`), with outcome `OutcomeGoto`, so a goto entry is never mistaken for this row's Stuck.

Tool calls:
1. `sed -n 78,122p internal/shedengine/status.go`: the full `HistoryEntry` fields (`At`, `LastAt`, folding), which my context showed only as grep lines.
2. A grep for `By:` and `Resume` in shedengine: to find whether a resume writes a history entry, which my context did not show.
3. `sed -n 255,300p internal/shedengine/goto.go`: the body of `RecordResumer`, to confirm a resume appends no history entry.
