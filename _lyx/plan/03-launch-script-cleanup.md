# Card 3 — reedengine: delete launch scripts when strands leave reed's state

**Edit:**
- `internal/reedengine/panebin.go`
- `internal/reedengine#Engine.removeStrandLocked`
- `internal/reedengine#Engine.Down`

**Create:**
- `plan:internal/reedengine#removeLaunchScripts` -> `func removeLaunchScripts(sh shell.Shell, stateDir string, guids []string)`
- `internal/reedengine/launchscript_cleanup_test.go`

**Uses:**
- `plan:internal/reedengine#launchScriptDir`
- `plan:internal/reedengine#launchScriptPath`
- `internal/reedengine#Engine.stateDir`
- `internal/reedengine#Engine.RemoveStrand`
- `internal/reedengine#Engine.ReplaceStrand`

**Intent:**
Hold the invariant that a launch script exists only while its GUID is in reed's persisted strand table.
GUIDs are unique per strand, so any path that forgets strands without deleting their scripts orphans those files permanently.
This card lands the deletion ahead of the write (card 4), so no build in the sequence writes scripts nothing deletes.

- `removeLaunchScripts` in `panebin.go` deletes `launchScriptPath(sh, stateDir, guid)` for each GUID.
  Deletion is best-effort: a missing file is silent, and any other `os.Remove` error is a `logger.Warn` naming `"strand"`, `"path"` and `"err"`, never returned.
- `removeStrandLocked`, the shared removal step behind both `RemoveStrand` and `ReplaceStrand`, calls `removeLaunchScripts(shell.ForGOOS(), e.stateDir(), toRemove)` after pruning `st.Strands`, so every GUID in a recursive subtree loses its script and surviving strands keep theirs.
  Its doc comment gains one sentence saying so.
- `Down`, right after it deletes `reed.json`, removes the whole `launchScriptDir(e.stateDir())` with `os.RemoveAll`; an error is a `logger.Warn` and never fails `Down`, which stays idempotent.
  `Down` leaves nothing for `Resume` to relaunch once the state file is gone, so no script has a strand left to belong to.

Tests in the new `launchscript_cleanup_test.go`, driving each public op through `newTestEngine` and the `e.tmux.execHook` fake the way `TestRemoveStrand_NeverKillsAPaneOutsideThisSession` does, with script files seeded by hand at `launchScriptPath(shell.ForGOOS(), e.stateDir(), guid)`:

- `RemoveStrand` of a leaf deletes that strand's script and leaves a sibling strand's script in place.
- `RemoveStrand` with `recursive` true on a parent deletes the scripts of the parent and every descendant, and leaves an unrelated strand's script in place.
- `ReplaceStrand` deletes the replaced strand's script and leaves a surviving strand's script in place.
  The replacement spec's display is `render.AnchorHidden`, so `needsLaunchOnAdd` skips the launch the fake's empty `split-window` answer would fail, and the test isolates the removal step.
- `RemoveStrand` of a strand that never had a script succeeds and logs no warning.
- `Down` removes both `reed.json` and the launch directory; the fake answers `list-sessions` with a sibling session name so `Down` skips the server teardown.
  A second `Down` with the directory already gone still succeeds.

**ImpactSummary:** `removeStrandLocked` is shared by `RemoveStrand` and `ReplaceStrand` and `Down` is one verb; each only gains a best-effort file delete that cannot change an operation's result or error.

**Commit:** 3: reedengine: delete launch scripts when strands leave reed's state
