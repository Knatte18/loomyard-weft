# Card 4 — ide-spawn-driven

**Create:**
- `internal/ideengine#SpawnDriven`
- `internal/ideengine/spawn_driven_hub_integration_test.go`

**Edit:**
- `internal/ideengine#Spawn`

**Uses:**
- `internal/vscode#TaskChain`
- `internal/vscode#TaskChainInteractive`
- `internal/vscode#TaskChainAttachOnly`
- `internal/fabricengine#ExcludeAnchoredDir`
- `internal/fabricengine#PathTracked`
- `internal/ideengine#CodeLauncher`

**Intent:**
Add `SpawnDriven` to `internal/ideengine/spawn.go`: the driven-pair variant of `Spawn` that batten calls once a child run's driver is up.
`l` is the caller's own location (batten's prime), exactly as `Spawn` takes it, and `slug` names the task worktree.
The caller decides which chain it wants; nothing here detects a driven pair, and nothing reads the seed's driver.

Steps, in order:
1. Resolve `worktreeDir := fabricengine.WorktreePath(l, slug)` and the title-bar color exactly as `Spawn` does (`fabricengine.PrimeName`, warn-and-degrade on error, `vscode.PickColor`).
   Extract that shared resolution into one unexported helper both functions call rather than duplicating it.
   The helper returns the worktree path, the color, the prime's name, and whether the prime's name resolved.
   It logs the prime-resolution warning itself, exactly once.
   `Spawn` keeps its prime-workspace branch keyed on the returned name and flag (resolved, and `slug` equals the prime's name), so `Spawn`'s behaviour and log output stay as they are; its `WriteConfig` call keeps passing `vscode.TaskChainInteractive`.
   `SpawnDriven` ignores the name and flag: it has no prime-workspace branch, since a batten slug is never the prime.
2. `fabricengine.PathTracked(worktreeDir, <AnchorRel>/.vscode/tasks.json)` (slash-separated, the bare `.vscode/tasks.json` for anchor `.`), returning its error wrapped.
   When tracked, the `.vscode/` directory belongs to the repo: skip both the exclude write and `vscode.WriteConfig`, leaving both files and the shared `info/exclude` untouched, log one `logger.Warn` naming the slug and the path, and go to step 4.
   Checking this first matters: git reports a directory with tracked content as not ignored, so an exclude write here would add a repo-wide line that serves nothing, and it would hide any new file under the repo's tracked `.vscode/` in every worktree.
3. When untracked, first `fabricengine.ExcludeAnchoredDir(worktreeDir, l.AnchorRel, ".vscode")`, logging the exclude path at `Info` when it changed.
   An error is returned wrapped with its step before anything is written or launched, so the pair's tree is never left with an unignored `.vscode/`.
   Then call `vscode.WriteConfig(worktreeDir, l.AnchorRel, slug, color, lyxPath, "", vscode.TaskChainAttachOnly)`, with `lyxPath` from `os.Executable()` degrading to `""` on error as `Spawn` does, and return its error wrapped.
4. `CodeLauncher(filepath.Join(worktreeDir, l.AnchorRel))`, returning its error: the bare task folder, the same directory batten runs the child's bootstrap in, so the `folderOpen` `reed attach` resolves the child's own reed session.

Never call `gitignore.Ensure` on this path: the child's loom run commits in that worktree.
Update the file's header comment to name both entry points.
`SpawnDriven`'s own doc comment states the chain, the overwrite rule, the tracked-file skip and the exclude step.

Tests in the new integration-tagged file, serial because they swap `CodeLauncher` (reuse `recordLauncher` from `spawn_hub_integration_test.go`), built on `hubforge.NewHub` plus `hubforge.AddPair`:
- For each anchor, `.` and `wts/some-task`, after `SpawnDriven`:
  - the launcher was called exactly once with `filepath.Join(fabricengine.WorktreePath(l, slug), l.AnchorRel)`, which also equals `lyxcwd.ResolveWorktree(worktreePath)`'s `AnchorPath()`;
  - the decoded `tasks.json` holds the single attach task;
  - `git status --porcelain` in the task worktree is empty;
  - `git check-ignore` reports the anchor's `.vscode/` ignored;
  - a custom line written into the shared `info/exclude` (resolved with `git rev-parse --git-path info/exclude`) before the call is still present after it.
- An untracked pre-existing interactive `tasks.json` (written by hand, not through `WriteConfig`'s interactive chain, which would edit `.gitignore`) is overwritten with the attach-only one, and a pre-existing `settings.json` stays byte-for-byte identical.
- With a committed `.vscode/tasks.json` in the task worktree:
  - the file stays byte-for-byte identical;
  - the shared `info/exclude` is byte-for-byte unchanged;
  - the launcher is still called once;
  - the captured log (`logger.SetOutput`) holds exactly one warning line naming the slug.

Plain `Spawn`'s existing tests stay unmodified and must still pass.
Every test reaching `SpawnDriven` stubs `CodeLauncher`; none starts a real `code` process.

**ImpactSummary:** Adds one exported entry point and a shared unexported resolution helper in `spawn.go`; `Spawn`'s observable behaviour and its callers are unchanged.
