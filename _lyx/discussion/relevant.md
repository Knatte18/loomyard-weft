# Relevant files

- `internal/testkit/scankit/scankit.go` `skipDirNames`, `collect`, `findModuleRoot` — the walk that opens the module root and so hashes the `.scratch` entry
- `internal/fabricengine/junction.go` `WireJunctionsWith`, `seedLyxJunction`, `seedGitExclude`, `excludePatternFor` — the wiring the scratch link joins and the link handling it copies
- `internal/fabricengine/gitexclude.go` `scratchDirName`, `seedScratchExclude`, `ExcludeAnchoredDir` — the scratch exclude whose trailing-slash form misses a link
- `internal/fabricengine/reconcile.go` `repairPairWiring`, `checkJunctionHealth` — where reconcile converges a pair, and the health check scratch stays out of
- `internal/fabricengine/fabric.go` `PairComplete`, `RecordsWorktree`, `WeftLyxDir` — a run gate that reads junction health, and the weft paths the target derives from
- `internal/preflight/preflight.go` — calls `fabricengine.Healthy`, a second gate on junction health
- `internal/fabricengine/status.go` `JunctionHealthy` — status reporting that stays scratch-blind
- `internal/cleanupcli/scratch.go` `scratchSegment` — lists the prime's `.scratch` and must keep working through the link
- `internal/cleanupcli/doc.go` — documents the scratch segment over the prime's `.scratch`
- `internal/lyxdirs/dirs.go` `DotLyxDirName` — the sole declarer of the `.lyx` name the target path uses
- `internal/fabriccli/clone.go` — the prime's clone-and-wire call into `WireJunctionsWith`
- `internal/fabricengine/destroy.go` — owns link removal and re-pointing, which the scratch step avoids
- `internal/fabricengine/unwire.go` `UnwireJunctions` — keeps the scratch exclude and, under D3, the scratch link
