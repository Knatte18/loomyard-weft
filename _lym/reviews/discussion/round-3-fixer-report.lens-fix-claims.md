```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the round-3 fix added to the decision record and support log holds against the code at HEAD.

No findings.

Every claim the fix added holds at HEAD:
- **Where the record sits:** `BeamReadyPath` puts `beam-ready.json` in `LandingDir`, which is anchored at `AnchorPath`, so the record lies under the beam's anchor (`internal/loomengine/config.go:102-115`).
- **The commit:** `MarkReady` commits it through `CommitAnchoredPaths` (`internal/beamcli/deps.go:251`). That commits in `RecordsWorktree` scoped under `AnchorRel` and never pushes (`internal/fabricengine/commitweftpaths.go`). The `<anchor>` prefix in the remote read matches this scoping.
- **The branch name:** `RecordsBranchName` appends the weft suffix, which gives `<beam branch>-weft` (`internal/fabricengine/branchname.go:11-13`).
- **One-repository hubs:** the fetch refspec maps `refs/lym/heads/*` to `refs/remotes/origin/*`, so `origin/<beam branch>-weft` resolves in both layouts (`pattern/PATTERN-single-repo-hub.md:17`).
- **The kept citation:** `internal/beamcli/cli.go:918-920` is the `beam ready` help text, which says the verb writes the ready record into `_lym/landing` and commits it. The citation is still relevant.

Tool calls:
- Read `internal/beamcli/cli.go:910-925`, `RecordsBranchName` and `internal/loomengine/config.go:98-112`. My context lacked the cited CLI lines, the branch-name function body and the `LandingDir` anchoring.
