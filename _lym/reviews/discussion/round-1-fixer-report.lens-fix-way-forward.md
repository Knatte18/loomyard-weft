```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the fix's ways forward with the fix-way-forward lens.

no findings

Each way forward the fix added can be reached and moves things forward:
- **Failed re-add or failed status sidecar write → `lym loom start`.** `ensureStatusStrand` adds the strand when none is found (`internal/loomcli/bootstrap.go:229`) and replaces it when the sidecar is stale. Its placement goes beside the runs strand because the runs strand now exists. The step runs before the run-lock probe, so a live pawl run doesn't block it.
- **Failed `ReplaceStrand` → `lym loom start` or `lym beam start`.** If the old strand is already gone, `lym loom start` adds the status strand back in the overview window. A later `lym beam start` then takes the loom status branch again, and D1 re-adds the strand. Either order makes progress.
- **Failed runs sidecar write → `lym beam start`.** On a rerun, a runs strand with a stale sidecar is replaced (`internal/orchcli/runs.go:84`), and the re-add still ran this time.
- **`AddStrandUnless` skip in the bootstrap.** The bootstrap adds only when `findStatusStrand` saw no status strand. I found no hidden-strand filter in reed's `Status`, so the skip is only reached through the race F3 describes. The other add then holds the strand, and that add writes its own sidecar.
- **D3's `reed.json` read.** `internal/fabriccli/anchordeps.go:379` already calls `reedengine.LoadState` on a `.lym` directory, so the overview can make the same read.

Tool calls:
1. `grep resolveStatusStrandAction` and reed's `Status` for hidden filtering: I needed to know when the bootstrap reaches its add, and my context lacked that code.
2. `grep LoadState(` callers: I needed to confirm a non-reed caller already reads `reed.json` from a `.lym` directory, and my context lacked that.
