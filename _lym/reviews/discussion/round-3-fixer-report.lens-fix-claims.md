```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked whether each bound, guarantee and code fact the fix added or rewrote holds at this worktree's HEAD.

**[LOW] D9's new Bound overstates what the baseline digest covers.**
- Changed text, D9: "Bound: … a baseline change is caught by the digest, not the snapshot."
- Problem: D9 says this without exception, but D7 leaves one case unguarded.
- Evidence: D7 says, unchanged, "a run already past Drift-Expect with none is checked without this guard". On such an in-flight run there is no digest. The snapshot also skips the baseline (`.lym/drift/` is ignored, and only the scratch capture directory is listed). So a baseline edit by the judge is caught by neither.
- Possible change: limit the claim to runs that have a digest, or name the exception.

**Code facts that hold:**
- "the worktree's `.lym` is a link into the weft side's `.lym`": `ls -la` shows `.lym -> …/drift-rework-weft/.lym`.
- "under the hub's board-level `.lym`, beside the run process record, under a name `internal/lymdirs` declares": `RunProcDir` (`internal/fabricengine/junctionnames.go:226-228`) joins `HubScratchDir(hub)`, which is the Bolt checkout's `.lym`, with `lymdirs.RunProcDirName`. Board-level directories are declared in `internal/lymdirs/dirs.go`. The new location sits outside both the warp and the weft checkout.
- The rest of the added text describes design to be built, such as the `Drift-Records` trailer and the newest-commit seam. It asserts nothing about current code that HEAD contradicts.

**Tool calls:**
1. grep for `RunProcDirName` and the `scope.json` write sites, plus a listing of the hub root: to confirm where the run process record lives and how `scope.json` is written. My context had only `lymdirs`' declarations.
2. Read `HubScratchDir` and `RunProcDir` in `junctionnames.go`, and listed the `driftengine` files that commit: to confirm that "the hub's board-level `.lym`" sits outside both checkouts.
