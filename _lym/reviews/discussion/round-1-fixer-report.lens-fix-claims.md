```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every sentence the fix added or rewrote that asserts a fact about the code against this worktree's HEAD.

no findings

Each claim and the code that supports it:
- **`AddStrandUnless` skip:** a strand already holding the unless-name makes it return that strand with `skipped = true` and no error. This happens before any add or reclaim, for live, dormant and hidden holders (`internal/reedengine/strand.go:599-604`). That supports the D1 skip and the rewritten Constraint. After the replace has removed the status strand, the only possible holder is one added concurrently, so "a concurrent `lym loom start` added the strand first" holds.
- **`ReplaceStrand` keeps the removal when its add fails:** it trims the failed add off the table and then runs `reconcileApplyPersistLocked` anyway (`internal/reedengine/replace.go:78-102`). Failures before the removal, in validation or session checks, leave the strand in place. The record's "may already be gone" fits both cases.
- **D3 reader:** reed's state directory is `filepath.Join(e.geom.AnchorPath, lymdirs.DotLymDirName)` (`internal/reedengine/lifecycle.go:36-37`), and `LoadState` reads `reed.json` in it. That matches "reed.json under the worktree's anchor `.lym` directory".
- **D3 window-name rule:**
  - A strand with no group, or one placed apart, opens its window named `strandLabel(s.Name)`, which is the role.
  - A strand in a new shared group calls `openOwnWindowLocked(st, s, group, group)`, whose third parameter is the window name (`internal/reedengine/spawn.go:164-198, 261`).
  - `planOwnWindowPlacement` places a group recorded as not shared apart (`ownwindow.go:215-219`).
  - The strand is alone in its group, so the "shared but full" case cannot happen.
- **F6 Constraint:** `LoadState` reads through `state.ReadJSON` with lock path `reed.json.lock`, which takes a shared read lock (`internal/reedengine/state.go:150-155`, `internal/state/state.go:96-107`).
- **F7 evidence:** `wireCLI` derives the reed geometry from `location` and sets `c.location` (`internal/orchcli/cli.go:203-209`). The three `orchcli.BeamUp` callers match the review's list.
- **Unless-name for the loom status role:** `statusStrandDisplayName = loomengine.LoomStatusStrandName`, which is `"loom-status"` (`internal/loomcli/bootstrap.go:24`, `internal/loomengine/driver.go:28`).

Tool calls:
1. `openOwnWindowLocked`'s signature and `replace.go` lines 78-110: I needed the window-name parameter and the replace failure path past the span I had read.
2. `loadOrInitStateLocked`, `statusStrandDisplayName` and `strand.go` lines 589-640: I needed the reed state directory, the status strand's name constant, and the order of the `AddStrandUnless` skip, which the earlier read stopped short of.
3. `stateDir` and `hubgeom`: I needed to confirm that reed's state directory is the anchor `.lym` directory.
