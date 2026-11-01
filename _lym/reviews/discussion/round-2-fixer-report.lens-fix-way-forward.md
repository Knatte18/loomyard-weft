```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, next step and test the fix introduced can be reached and leads to progress.

**Findings**

1. **LOW.** Changed text: D2's `render [<worktree>]` bullet and its refusal ("a name that is none of the hub's warp worktrees, as `lym fabric list` lists them, is refused", with the way forward `lym fabric list`).
   - Summary: the way forward lists worktree paths, but `render` takes a worktree name. The reader has to work out the name from each path's base name, and the record never says so.
   - Evidence: `internal/fabriccli/fabric.go:170-178` says the verb "outputs warp worktree paths only". `runList` returns `WorktreeEntry{Path, Head, Branch, Main, Prunable}` (`internal/fabricengine/worktreelist.go:17-23`), which has no name field. `lymcwd.NamedWorktree(hubPath, name)` (`internal/lymcwd/lymcwd.go:112`) takes a bare name.

2. **LOW.** Changed text: the same D2 bound, "none of the hub's warp worktrees, as `lym fabric list` lists them".
   - Summary: `lym fabric list` is `git worktree list --porcelain`, which also lists prunable entries whose directory is gone. A name that matches only a prunable entry therefore passes the bound, and `render` then writes into a directory that does not exist. That refusal or error is not named, so the bound does not fully hold as the way forward implies.
   - Evidence: `internal/fabricengine/worktreelist.go:28-34` returns the porcelain list unfiltered, and `WorktreeEntry` carries `Prunable`.

The other ways forward the fix introduced check out:
- D3's exclude-list refusal, now covering `.heddle`, names `warp_exclude` and can be carried out.
- D6's log-only warning names `lym heddle render`, which can be run in that worktree or by name.
- The new D3 and D5 acceptance cases (rename, no `.heddle` left behind, a concurrent Bolt fast-forward, `/.heddle` seeded once) and the D6 `ReconcileOrigin` case can all be built from the states they name.
- The D2 refusal acceptance case can be built.
- Lock order holds where I checked: `CloneHub` (`internal/fabricengine/clone.go`) takes no Bolt lock before the point where "made" would fire, and `ReconcileOrigin`'s code-branch move runs under the prime lock (`originreconcilemoves.go:336-341`), not the Bolt write lock.

Tool calls:
- Read `runList` and found `CloneHub`: my context lacked what `lym fabric list` outputs and where `CloneHub` lives.
- Read `Topology.List` and grepped `CloneHub` for locks: my context lacked the list's source and `CloneHub`'s lock use.
- Read `WorktreeEntry` and `List`: my context lacked the entry fields, to judge whether the listed paths map to the names `render` takes.
