# Card 9 — reed-env-injection

**Edit:**
- `internal/reedengine/panebin.go`
- `internal/reedengine/spawn.go`
- `internal/reedengine/panebin_test.go`

**Uses:**
- `internal/agentname/agentname.go`
- `internal/shell/shell.go`

**Intent:**
Every process reed starts sees its own name and its worktree's parent.
`composePaneLaunchLine` takes the strand's name and the told `Geometry.ParentName`, and the launch script it composes exports `LYX_STRAND_NAME=<full name>` and, when the parent is non-empty, `LYX_PARENT=<parent>`, each through the dialect's `ExportEnv` with the key from `agentname`, chained after the pane-binary prelude and before the launch command.
The exports do not depend on the executable lookup: when `executablePath` fails the prelude is still dropped, but the two exports stay.

`launchStrandLocked` is the one launch every path funnels through — add, the hidden-to-visible surface, `--if-absent` relaunch, replace and every per-strand replay inside `Resume` — so a resumed process after a server rebirth sees the same values, since the name is the stored birth attribute and the parent is re-told from the seed.
The panebin file doc and `launchStrandLocked`'s comment name the two exports beside the `LYX_BIN` one.

`panebin_test.go`: for both dialects, the composed line carries the name export, carries the parent export only when a parent is told, keeps the exports when the executable lookup fails, and orders prelude, exports, then the command.

**ImpactSummary:** Adds two env exports to every strand's launch script; launch and resume paths are otherwise unchanged.

**Commit:** 9: reed: export LYX_STRAND_NAME and LYX_PARENT into every strand
