# Card 1 — hub-workspace-regenerate

**Edit:**
- `internal/fabricengine#HubWorkspacePath` -> `func HubWorkspacePath(l *lyxcwd.Location) string`
- `internal/fabricengine#hubWorkspaceExt`
- `internal/fabricengine/doc.go`
- `internal/ideengine#Spawn`
- `internal/ideengine#resolveSpawnTarget`
- `internal/fabricengine#TestHubWorkspacePathAndFolders`
- `internal/fabricengine#TestWriteHubWorkspace`
- `internal/fabricengine#TestWriteHubWorkspace_ReadErrorFailsWithReadCause`
- `internal/ideengine#resetPrimeEditorState`
- `internal/ideengine#checkPrimeWritesHubWorkspace`
- `internal/ideengine#checkTaskSlugOpensBareFolder`
- `internal/ideengine#checkPrimeSplicesSettingsAndRegenerates`
- `internal/ideengine#checkPrimeSettingsReadFailure`
- `internal/ideengine#checkPrimeWorkspaceSurvivesTopologyVerbs`
- `internal/ideengine#checkSkipsTrackedVSCode`
- `PATTERN.md`
- `pattern/PATTERN-hub-containment.md`

**Delete:**
- `internal/fabricengine#HubWorkspaceFolder`
- `internal/fabricengine#HubWorkspaceFolders`
- `internal/fabricengine#WriteHubWorkspace`
- `internal/ideengine#writePrimeWorkspace`
- `internal/ideengine#writeWorkspaceFile`
- `internal/fabricengine#TestHubWorkspaceFolders_OmitsMissingDirectories`
- `internal/fabricengine#TestWriteHubWorkspace_RefusesEscapingLaunchersSymlink`

**Create:**
- `plan:internal/fabricengine#RegenerateHubWorkspace` -> `func RegenerateHubWorkspace(rec *Mutations, l *lyxcwd.Location) error`
- `plan:internal/fabricengine#hubWorkspaceFolders` -> `func hubWorkspaceFolders(l *lyxcwd.Location, entries []WorktreeEntry) ([]vscode.WorkspaceFolder, error)`
- `plan:internal/fabricengine#writeHubWorkspace` -> `func writeHubWorkspace(l *lyxcwd.Location, content []byte) (bool, error)`
- `plan:internal/fabricengine#HubWorkspaceLockPath` -> `func HubWorkspaceLockPath(hubPath string) string`
- `plan:internal/fabricengine#hubWorkspaceLockWait` -> `var hubWorkspaceLockWait = 5 * time.Second`
- `plan:internal/fabricengine#SetHubWorkspaceLockWaitForTest` -> `func SetHubWorkspaceLockWaitForTest(t *testing.T, wait time.Duration)`
- `plan:internal/fabricengine#hubWorkspaceLockedHook` -> `var hubWorkspaceLockedHook func()`
- `plan:internal/fabricengine#SetHubWorkspaceLockedHookForTest` -> `func SetHubWorkspaceLockedHookForTest(t *testing.T, hook func())`
- `plan:internal/fabricengine_test#TestRegenerateHubWorkspace` -> `func TestRegenerateHubWorkspace(t *testing.T)`

**Uses:**
- `internal/fabricengine#List`
- `internal/fabricengine#BoardDir`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#hubRel`
- `internal/fabricengine#NewMutations`
- `internal/fabricengine#KindFileWritten`
- `internal/fabricengine#KindDirCreated`
- `internal/fabricengine#SetGCAutoThresholdForTest`
- `internal/vscode#BuildWorkspace`
- `internal/vscode#ReadSettings`
- `internal/vscode#RelativeSettingKeys`
- `internal/lock#AcquireWriteLockWithin`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`
- `internal/testkit/locationkit#Location`
- `internal/testkit/logcapture#Capture`
- `internal/fabricengine#CodeWorktrees`

**Intent:**
Build the one regeneration function D3 names, owned by `fabricengine`, and move the prime spawn onto it (D1, D2, D3, D5).
`fabricengine` now imports `internal/vscode`; `internal/vscode` imports only `logger`, `lyxcwd` and `proc`, so no cycle forms, and the "imports no vscode package" comment goes with `HubWorkspaceFolder`.

Geometry (D1).
`HubWorkspacePath(l)` returns `<hub>/<l.RepoName>.code-workspace`, `l.HubPath` joined with `l.RepoName + hubWorkspaceExt`, so the loomyard hub's file is `loomyard.code-workspace`.
Its doc and `hubWorkspaceExt`'s doc stop calling it the prime's file under `_launchers`.
`HubWorkspaceLockPath(hubPath)` returns `<HubScratchDir(hubPath)>/hub-workspace.lock`, i.e. `<hub>/_board/.lyx/hub-workspace.lock` (D5); it is exported for the external test package's held-lock row, as `TeardownLockPath` is.

Folders (D2).
`hubWorkspaceFolders(l, entries)` takes the parsed `git worktree list` entries of the warp repo and returns, in order:
the `Main` entry's anchor dir `<entry path>/<l.AnchorRel>`, named after `entry.Branch`, or after the directory's base name when `entry.Branch` is `(detached)` (the parser's detached spelling);
then `BoardDir(l.HubPath)` named `Bolt`, only when it exists as a directory;
then every other entry's anchor dir, named after the entry directory's base name (the slug), sorted by name in byte order (`sort.Strings`).
A non-main entry is skipped when it is `Prunable`, when its directory's parent is not `l.HubPath` (a linked worktree added outside the hub, logged at Debug, so the file references hub-inward folders only per PATTERN-hub-containment), when `lyxcwd.ResolveWorktree` fails on its path (logged at Debug with the path and error, so one half-created pair never drops the others), or when its anchor dir does not exist.
Weft worktrees are never in the list, since the entries are the warp repo's; local pairs and pairs forked from pairs are ordinary entries and are listed.
A folder whose directory does not exist is left out (the prime's too); a stat error other than not-exist and a `filepath.Rel` error are returned.
Each `Path` is relative to `l.HubPath` (the file's directory) and slash-separated, e.g. `loomyard` for a prime directory `loomyard` at anchor `.`, `_board`, `a-task/wts/x`; the display name is the branch, the path is always the directory.
No entry lists the retired `_portals` folder.

Write.
`writeHubWorkspace(l, content)` replaces `WriteHubWorkspace`: it opens an `os.Root` at `l.HubPath`, reads `HubWorkspacePath(l)`'s hub-relative name through it, returns `false, nil` when the bytes are equal, returns a read error other than not-exist as `read workspace file …`, and otherwise writes the file in place through `root.WriteFile` (D5: no temp file, no atomic replace) and returns `true`.
It no longer creates `_portals/<AnchorRel>` or any parent directory, since the file sits directly in the hub dir.
A symlink planted at the workspace path that resolves outside the hub is refused by the `os.Root` read and write, and nothing lands at its target (PATTERN-fabric-write-containment as declared).
A truncated or hand-edited file differs in bytes and is rewritten whole.

Regeneration (D3, D5).
`RegenerateHubWorkspace(rec, l)`:
1. `List(l.AnchorPath())`; a list error, or no `Main` entry, returns an unresolvable-prime error and writes no workspace file.
2. Reads `vscode.ReadSettings(<prime anchor dir>)`; an error returns `read prime settings: …` (the wording `checkPrimeSettingsReadFailure` pins) and writes no workspace file.
   When `vscode.RelativeSettingKeys` is non-empty it logs one Warn that those keys now resolve against the hub directory.
3. When `BoardDir(l.HubPath)` exists as a directory, creates `HubScratchDir` through an `os.Root` at the hub when absent (appending `KindDirCreated` for it when this call minted it), then takes `HubWorkspaceLockPath` exclusive with `lock.AcquireWriteLockWithin(path, hubWorkspaceLockWait)`, appending `KindFileWritten` for the lock file when this call brought it into being (as `lockPairTeardown` does).
   A wait that runs out returns an error naming the lock path and writes no workspace file; the lock is a leaf lock, taken after whatever lock the caller holds, with nothing taken under it.
   With no `_board` it takes no lock, creates no directory and lists no Bolt.
4. Under the lock, after calling `hubWorkspaceLockedHook` when it is non-nil (nil in production; a test seam only), it re-lists the worktrees, computes `hubWorkspaceFolders`, builds the bytes with `vscode.BuildWorkspace`, and calls `writeHubWorkspace`; a write error is returned, and a write appends `KindFileWritten` for `HubWorkspacePath(l)`.
   An unchanged file appends nothing.
`hubWorkspaceLockWait` is a package var so `SetHubWorkspaceLockWaitForTest` (in `export_test.go`, beside `SetGCAutoThresholdForTest`, restoring it in `t.Cleanup`, non-parallel callers only) shortens it; `SetHubWorkspaceLockedHookForTest` sets and restores `hubWorkspaceLockedHook` the same way.

Prime spawn.
`Spawn` for the prime calls `fabricengine.RegenerateHubWorkspace(fabricengine.NewMutations(l.HubPath), l)`, discards the recorder, returns any error unlaunched, and launches `HubWorkspacePath(l)`.
An unresolvable prime keeps degrading to the bare folder through `resolveSpawnTarget`'s Warn, as today; only failures past prime resolution (unreadable settings, lock timeout, failed write) reach the caller.
`writePrimeWorkspace` (and its double write, which existed only for `_portals`) and `writeWorkspaceFile` are deleted; `spawn.go`'s file comment, `Spawn`'s doc and `resolveSpawnTarget`'s doc name the hub file `<hub>/<RepoName>.code-workspace` listing the prime, Bolt and every live pair.

Docs.
`internal/fabricengine/doc.go` states the file's path, folders and order, that it lists every task pair of the hub with no per-umbrella partition, that lyx owns it and every regeneration overwrites edits (settings belong in the prime's `.vscode/settings.json`), and the lock.
`PATTERN.md`'s two entry lines become the declared text:

```
- `PATTERN-fabric-write-containment` — Writing under `_launchers`, or the hub workspace file, from `fabricengine`: via an `os.Root` at the hub, never raw `os.MkdirAll`, `os.WriteFile` or `fslink`.
- `PATTERN-hub-containment` — Linking hub-level containers or writing the hub workspace file: from the hub only, never into a worktree. — [background](pattern/PATTERN-hub-containment.md)
```

`pattern/PATTERN-hub-containment.md`'s workspace bullet becomes: the hub workspace file sits directly in the hub dir, references hub-inward folders only and is never written into a worktree; its `_portals` lines are Card 5's.

Tests.
`TestHubWorkspacePathAndFolders` (untagged, `locationkit` locations with `RepoName` set, synthetic dirs, hand-built `WorktreeEntry` slices) pins the path at a root and a nested anchor and gains rows for: order `main`, `Bolt`, `a-task`, `b-task` with entries given `b-task` first; a detached prime named after its directory; a pair whose `.git` is a gitfile naming a missing git dir skipped while its sibling is listed; a prunable entry and a missing anchor dir skipped; an entry whose directory sits outside the hub skipped; a missing prime anchor dir left out with the pairs still listed; a regular file planted at a component of a pair's anchor path (`<pair>/wts` as a file at anchor `wts/x`) returning the stat error; no `_board` dir lists no `Bolt`; slash-separated relative paths.
Every listed pair directory in these rows carries a `.git` entry `lyxcwd.ResolveWorktree` accepts (a `.git` directory as `dotgit.FindRoot` reads it), since a bare directory is skipped as unresolvable.
`TestHubWorkspaceFolders_OmitsMissingDirectories` is deleted, its case now a row there.
`TestWriteHubWorkspace` drives `writeHubWorkspace`: first write true, identical rewrite false with bytes unchanged, changed rewrite true, a truncated file rewritten whole, and a row planting a symlink at the workspace path that points outside the hub, which must error with the outside target untouched (it replaces the deleted `TestWriteHubWorkspace_RefusesEscapingLaunchersSymlink`, whose `_launchers` parent no longer exists).
`TestWriteHubWorkspace_ReadErrorFailsWithReadCause` calls `writeHubWorkspace` with a directory planted at the new path.
`TestRegenerateHubWorkspace` (integration, package `fabricengine_test`, one `CopyHub` per subtest) covers: a changed file records exactly one `KindFileWritten` at the file and a second call records none; with `_board/.lyx` removed first, the call records `KindDirCreated` for `HubScratchDir` and `KindFileWritten` for the lock file; a Location whose worktree is not a git checkout returns the unresolvable-prime error with no file written; with `SetHubWorkspaceLockWaitForTest` at a few milliseconds and the test holding `HubWorkspaceLockPath` through `lock.AcquireWriteLock`, the call returns the lock error promptly with no file written; with `_board` removed, `_board` and `HubWorkspaceLockPath`'s directory stay absent after the call (Bolt's omission is the untagged row's); with `SetHubWorkspaceLockedHookForTest` set to a hook that adds a pair by raw `git worktree add`, the file lists that pair, so the listing runs under the lock; a prime `.vscode/settings.json` holding `{"x": "./rel"}` logs one Warn naming the key and the hub directory, captured with `logcapture.Capture`.
The held-lock, hook and log-capturing subtests are non-parallel.
The unreadable-settings failure stays pinned through the spawn by `checkPrimeSettingsReadFailure`.
The ideengine checks switch to `HubWorkspacePath(l)` and to the new folders: `checkPrimeWritesHubWorkspace` expects the prime's branch name (read with `gitkit.CurrentBranch`) at the prime's anchor dir, then `Bolt`, then the live pairs earlier scenario steps added, sorted by slug, each at its anchor dir (it reads the slugs from `fabricengine.CodeWorktrees` rather than pinning a count), with no `_portals` folder and no `_portals` directory created.
`checkPrimeSettingsReadFailure` gains a second half: a directory planted at `HubWorkspacePath(l)` (settings readable again) makes `Spawn` return the `read workspace file` error with no launch, so a failure past prime resolution other than unreadable settings reaches the caller through `Spawn` too.
The other checks only change the path call.

**ImpactSummary:** fabricengine's workspace surface is replaced and its sole consumer, ideengine's prime spawn, moves onto it; callers are those two packages and their tests only.

**Commit:** 1: one hub workspace file at <hub>/<RepoName>.code-workspace, regenerated by fabric
