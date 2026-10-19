# Card 1 — hub-workspace-file

**Edit:**
- `internal/fabricengine#HubWorkspacePath` -> `func HubWorkspacePath(l *lyxcwd.Location) string`
- `internal/fabricengine#TestHubWorkspacePathAndFolders`
- `internal/fabricengine#TestWriteHubWorkspace`
- `internal/fabricengine#TestWriteHubWorkspace_ReadErrorFailsWithReadCause`
- `internal/fabricengine/doc.go`
- `internal/vscode/config.go`
- `internal/ideengine#TestSpawnScenario`
- `internal/ideengine#Spawn`
- `internal/ideengine#resetPrimeEditorState`
- `internal/ideengine#checkPrimeWritesHubWorkspace`
- `internal/ideengine#checkTaskSlugOpensBareFolder`
- `internal/ideengine#checkPrimeSplicesSettingsAndRegenerates`
- `internal/ideengine#checkPrimeSettingsReadFailure`
- `internal/ideengine#checkPrimeWorkspaceSurvivesTopologyVerbs`
- `internal/ideengine#checkSkipsTrackedVSCode`

**Delete:**
- `internal/fabricengine#HubWorkspaceFolder`
- `internal/fabricengine#HubWorkspaceFolders`
- `internal/fabricengine#WriteHubWorkspace`
- `internal/fabricengine#TestHubWorkspaceFolders_OmitsMissingDirectories`
- `internal/fabricengine#TestWriteHubWorkspace_RefusesEscapingLaunchersSymlink`
- `internal/ideengine#writePrimeWorkspace`
- `internal/ideengine#writeWorkspaceFile`

**Create:**
- `plan:internal/fabricengine#RegenerateHubWorkspace` -> `func RegenerateHubWorkspace(rec *Mutations, l *lyxcwd.Location) (string, error)`
- `plan:internal/fabricengine#regenerateHubWorkspace` -> `func regenerateHubWorkspace(rec *Mutations, l *lyxcwd.Location, lockWait time.Duration) (string, error)`
- `plan:internal/fabricengine#hubWorkspaceLockWait` -> `const hubWorkspaceLockWait = 5 * time.Second`
- `plan:internal/fabricengine#HubWorkspaceLockPath` -> `func HubWorkspaceLockPath(hubPath string) string`
- `plan:internal/fabricengine#lockHubWorkspace` -> `func lockHubWorkspace(rec *Mutations, hubPath string, wait time.Duration) (func(), error)`
- `plan:internal/fabricengine#hubWorkspaceFolders` -> `func hubWorkspaceFolders(l *lyxcwd.Location, primeName string) ([]vscode.WorkspaceFolder, error)`
- `plan:internal/fabricengine#writeHubWorkspace` -> `func writeHubWorkspace(l *lyxcwd.Location, content []byte) (bool, error)`
- `plan:internal/fabricengine#RegenerateHubWorkspaceWithinForTest` -> `func RegenerateHubWorkspaceWithinForTest(rec *Mutations, l *lyxcwd.Location, wait time.Duration) (string, error)`
- `plan:internal/fabricengine#hubWorkspaceBeforeLockHook` -> `var hubWorkspaceBeforeLockHook func()`
- `plan:internal/fabricengine#SetHubWorkspaceBeforeLockHookForTest` -> `func SetHubWorkspaceBeforeLockHookForTest(t *testing.T, hook func())`
- `plan:internal/fabricengine_test#TestRegenerateHubWorkspace` -> `func TestRegenerateHubWorkspace(t *testing.T)`
- `plan:internal/fabricengine#TestWriteHubWorkspace_RefusesEscapingSymlink` -> `func TestWriteHubWorkspace_RefusesEscapingSymlink(t *testing.T)`
- `plan:internal/ideengine#checkPrimeWriteFailure` -> `func checkPrimeWriteFailure(t *testing.T, h *hubforge.Hub)`

**Prosa:**
- `README.md`
- `PATTERN.md`
- `pattern/PATTERN-hub-containment.md`
- `docs/overview.md`

**Uses:**
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#readBranch`
- `internal/fabricengine#List`
- `internal/fabricengine#WorktreePath`
- `internal/fabricengine#BoardDir`
- `internal/fabricengine#BoardDirName`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#NewMutations`
- `internal/fabricengine#KindFileWritten`
- `internal/vscode#BuildWorkspace`
- `internal/vscode#ReadSettings`
- `internal/vscode#RelativeSettingKeys`
- `internal/lock#AcquireWriteLockWithin`
- `internal/lock#AcquireWriteLock`
- `internal/ideengine#resolveSpawnTarget`
- `internal/ideengine#recordLauncher`
- `internal/ideengine#primeOf`
- `internal/ideengine#primeSettingsPath`
- `internal/ideengine#checkPrimeNameFailureOpensBareFolder`
- `internal/fabricengine#mkdirAll`
- `internal/testkit/locationkit#Location`
- `internal/testkit/logcapture#Capture`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`
- `internal/hubforge#AddPairWith`

**Intent:**
Replace the prime's workspace file under `_launchers` with one hub workspace file, `<hub>/<RepoName>.code-workspace`, and one fabric function that writes it (D1, D2, D3, D5).

`HubWorkspacePath(l)` returns `filepath.Join(l.HubPath, l.RepoName+".code-workspace")`; it is the only declaration of the path.
`RegenerateHubWorkspace(rec, l)` calls `regenerateHubWorkspace(rec, l, hubWorkspaceLockWait)` and returns the file's path.
`regenerateHubWorkspace` runs, in order:
1. Resolve the prime with `PrimeName(l)`; a failure returns its error and writes nothing.
2. Read the prime's settings with `vscode.ReadSettings(<hub>/<prime>/<AnchorRel>)`; a failure returns an error wrapped as `read prime settings: ...` and writes nothing.
   When `vscode.RelativeSettingKeys` finds keys, log one Warn naming them and saying they now resolve against the hub directory, where the workspace file sits.
3. When `BoardDir(l.HubPath)` exists, take the hub workspace lock through `lockHubWorkspace(rec, l.HubPath, lockWait)`.
   It creates `HubScratchDir` through an `os.Root` at the hub when it is absent (as the hub reconcile lock creates its dir) and takes `lock.AcquireWriteLockWithin(HubWorkspaceLockPath(hub), wait)`.
   It appends `KindFileWritten` for the lock file only when this call created it, the way `Topology.lockPairTeardown` records its lock file.
   A wait that runs out returns an error naming the lock path and the wait, and writes nothing.
   On a hub with no `_board`, take no lock, create no directory and list no Bolt.
   `HubWorkspaceLockPath(hub)` is `<HubScratchDir(hub)>/hub-workspace.lock`.
   The lock is a leaf lock: nothing is locked while it is held.
   `hubWorkspaceBeforeLockHook`, nil in production and set only by a non-parallel test through `SetHubWorkspaceBeforeLockHookForTest`, runs just before the lock is taken.
4. List the folders with `hubWorkspaceFolders(l, prime)`.
   Then build the bytes with `vscode.BuildWorkspace`, splicing the settings verbatim, and write them with `writeHubWorkspace`.
   Append `KindFileWritten` at the file's path only when it wrote.

`hubWorkspaceFolders` returns, in order, each path relative to the hub and slash-separated:
1. The prime's anchor dir, named after the prime's branch from `readBranch`, or after the prime's directory name when `readBranch` reports a detached `HEAD`.
2. `_board`, named `Bolt`.
3. Every other code worktree from `List(l.AnchorPath())`, skipping the main and prunable entries.
   Each is resolved with `lyxcwd.ResolveWorktree` for its anchor dir.
   An entry that fails to resolve is skipped with a Debug log naming its path and the error, so one half-made pair never drops the others; `CodeWorktrees` is left as it is.
   An entry whose cleaned worktree path is not a direct child of `l.HubPath` is skipped with a Debug log naming its path, like an entry that fails to resolve, so the file references hub-inward folders only.
   Each is named after its directory base name, the slug, and the pairs are sorted by name in byte order.
A folder whose directory does not exist is left out, a stat error other than not-exist is returned, and weft worktrees are never listed because `List` reads the warp repository only.
Local pairs and pairs forked from pairs are listed like any other.

`writeHubWorkspace(l, content)` writes the file in place through an `os.Root` at the hub, with no temp file and no `_portals` or `_launchers` directory.
It reads the existing bytes through the same root and reports `false` without writing when they are equal.
A read error other than not-exist is returned without writing, so a symlink planted at the workspace path that escapes the hub is refused.
It replaces `WriteHubWorkspace`, whose only production caller is `ideengine`.
`HubWorkspaceFolder` and `HubWorkspaceFolders` go: the folders are `vscode.WorkspaceFolder` values now that `fabricengine` imports `internal/vscode`, and the comment saying it imports no vscode package goes with them.
`RegenerateHubWorkspaceWithinForTest` in `export_test.go` exposes `regenerateHubWorkspace` with a short wait, so the lock-timeout row does not wait out 5 seconds.

`ideengine.Spawn`, for a slug that `resolveSpawnTarget` resolved as the prime, calls `fabricengine.RegenerateHubWorkspace(fabricengine.NewMutations(l.HubPath), l)`, discards the recorder and launches the returned path.
It returns the function's error unchanged, so an unreadable-settings, lock-timeout or failed-write failure fails the spawn.
An unresolvable prime keeps degrading to the bare folder with `resolveSpawnTarget`'s Warn, as today.
`writePrimeWorkspace` and `writeWorkspaceFile`, with their double write, go.
The `spawn.go` header comment and `Spawn`'s doc comment name the new file.

Tests:
- `TestHubWorkspacePathAndFolders` becomes a path-only check: for both anchors and a location with `RepoName` set, `HubWorkspacePath` is `<hub>/<RepoName>.code-workspace`.
  Its doc comment and `//testtiming:keep` reason name the path alone, and the folder geometry moves to `TestRegenerateHubWorkspace`.
- `TestWriteHubWorkspace` and `TestWriteHubWorkspace_ReadErrorFailsWithReadCause` call `writeHubWorkspace` with the new path.
  They keep the first-write, identical-rewrite and changed-rewrite assertions and the read-error assertion, and assert that no `_portals` or `_launchers` directory is created.
- `TestWriteHubWorkspace_RefusesEscapingSymlink` replaces `TestWriteHubWorkspace_RefusesEscapingLaunchersSymlink`.
  It plants the escaping symlink at the workspace file path itself, and asserts the write fails and the symlink's target directory stays empty.
  It is the read-path refusal: the root's read refuses the symlink before any write.
- `TestHubWorkspaceFolders_OmitsMissingDirectories` goes; its behavior is row 5 of `TestRegenerateHubWorkspace`.
- `TestRegenerateHubWorkspace` (integration, external package) uses `hubforge.CopyHub` hubs.
  Rows whose arrangements do not conflict may share one hub.
  Rows that assert a failure writes no file first remove the workspace file, which `CloneHub` writes from card 2 on, and assert it stays absent.
  A row expecting a call to write after the pair set changed changes the pair set with a raw `git worktree add`, never through `hubforge.AddPair` or another verb, because from card 2 on the verb regenerates by itself and the following call finds identical bytes.
  The rows:
  1. Pairs `b-task` then `a-task` give folders `<prime branch>`, `Bolt`, `a-task`, `b-task` in that order, each resolving to its anchor dir relative to the file's directory, with no backslash.
     The rows run at the root anchor and a nested one.
  2. A pair added with `AddPairWith` and `AddLocal` (local) is listed.
  3. A detached prime (`git switch --detach` in the prime) is listed under its directory name.
  4. `_board` moved aside gives no `Bolt` folder, and `_board` stays absent after the call.
  5. The three skip exits:
     - At the root anchor, a pair whose worktree dir is deleted by hand reads as prunable and is left out.
     - At a nested anchor, a pair whose anchor subdir alone is deleted is left out.
     - A warp worktree added by `git worktree add` to a directory outside the hub is left out.
     The other pair stays listed in each.
  6. A pair whose `.git` file is overwritten with garbage so `lyxcwd.ResolveWorktree` fails on it is skipped while the other pair stays listed.
  7. The mutation record:
     - A call on an unchanged hub records no `KindFileWritten` for the file.
     - A changed folder list, arranged by adding a pair with a raw `git worktree add` after a first call has written the file, as row 12 does, so no fabric verb regenerates, a changed prime `settings.json` alone, and a truncated prefix written over the file each record `KindFileWritten` at the file's path.
       The truncated case leaves the whole content.
  8. With `HubScratchDir` removed, a call recreates it and records `KindFileWritten` at `HubWorkspaceLockPath(hub)`; a second call does not record the lock file.
  9. A directory planted at the prime's `.vscode/settings.json` returns an error containing `read prime settings` and writes no workspace file.
  10. A location whose worktree is not a git checkout returns the prime-resolution error and writes no workspace file.
  11. A lock held through `lock.AcquireWriteLock(HubWorkspaceLockPath(hub))` makes `RegenerateHubWorkspaceWithinForTest` with a 50ms wait return an error naming the lock, within a bounded time, and write no workspace file.
  12. With the lock held through `lock.AcquireWriteLock(HubWorkspaceLockPath(hub))`, a goroutine starts `RegenerateHubWorkspaceWithinForTest` with a wait longer than the step.
      The row sets `SetHubWorkspaceBeforeLockHookForTest` to signal a channel, and waits for that signal, so the goroutine has reached the lock before `b` exists; the row is serial, because the hook is process-global.
      While the lock is held, pair `b` is created with a raw `git worktree add`, so no verb regenerates.
      The lock is released, the goroutine is joined, and the file lists `b`, which a regeneration that listed the folders before taking the lock would miss.
      No pair is added from a goroutine.
  13. Settings holding `"x": "./rel"` log a Warn naming the key and the hub directory, through `logcapture.Capture`; this row is not parallel.
- The ide checks in `spawn_hub_integration_test.go` use `HubWorkspacePath(l)`:
  - `checkPrimeWritesHubWorkspace` asserts the launch argument is the hub file and the first two folders are the prime's branch at its anchor dir and `Bolt` at `_board`.
    It also asserts a folder for each pair the scenario added before it, and no `_portals` folder in the file.
  - `checkTaskSlugOpensBareFolder` asserts a task spawn leaves the workspace file's bytes (or its absence) unchanged, in place of asserting no file exists.
  - `checkPrimeSettingsReadFailure` keeps the `read prime settings` cause and the no-file assertion.
  - `checkPrimeWriteFailure` places a file of different bytes at `HubWorkspacePath` with mode `0o444`, so `root.WriteFile` fails on the existing file, and its cleanup removes the file so the later prime-spawning steps write afresh.
    It skips when `os.Geteuid() == 0`, where mode bits refuse nothing.
    It asserts `Spawn` returns the write error, `CodeLauncher` is not called and the file keeps its bytes.
    `TestSpawnScenario` runs it as a step after the settings-failure step.
    A lock-timeout spawn row is not planned: the timeout reaches `Spawn` through the same returned error as the write failure, and a spawn-level row would wait out the 5-second production wait.
  - The other checks only take the new path argument.

Docs in this card:
- `README.md`'s hub tree adds `<RepoName>.code-workspace` at the hub level, and its `_launchers/` line names launcher scripts only.
- `PATTERN.md`'s `PATTERN-fabric-write-containment` and `PATTERN-hub-containment` lines take the decision record's `## PATTERN changes` text.
- `pattern/PATTERN-hub-containment.md`'s workspace bullet says the hub workspace file sits directly in the hub dir, references hub-inward folders only and is never written into a worktree.
- `docs/overview.md`'s topology tree moves the workspace file to `<hub>/<RepoName>.code-workspace`, and its ide bullet lists the prime, `Bolt` and every task pair.
- `internal/fabricengine/doc.go` states that the hub workspace file lists every task pair of the hub, that no per-umbrella partition exists, and that the file is lyx-owned, with settings belonging in the prime's `.vscode/settings.json`.
- `internal/vscode/config.go`'s Limitation paragraph names the hub workspace file at `<hub>/<RepoName>.code-workspace` and the hub directory as what relative settings resolve against.

**ImpactSummary:** Re-signs `HubWorkspacePath` and drops three exported workspace symbols, whose only callers are `ideengine` and the tests listed here; the folder list and file location change for `lyx ide spawn <prime>`.
