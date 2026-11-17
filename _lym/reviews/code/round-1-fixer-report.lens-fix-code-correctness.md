Lens fix-code-correctness.

1. LOW: `hubStopWorktrees` dropped an unwired pair (`if !wired { continue }` in `internal/beamcli/hubstop.go`), so its reed session was never downed and no failure named it.
   Everything else held: `detachedStopCommand`, the `runHubStop` lift skip, the `hubStopWorktrees` error return, `beamDown` and `endLiveDobby`, the `TestRunHubStop` rewrite, the integration test and the refusal row.

Tool calls: none.
