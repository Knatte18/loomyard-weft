# Card 8 — workspace-content-seam

**Create:**
- `internal/fabricengine#WorkspaceContent`
- `internal/fabricengine#InstallWorkspaceContent`
- `internal/fabricengine#workspaceMembers`

**Edit:**
- `internal/vscode#BuildWorkspace` -> `func BuildWorkspace(folders []WorkspaceFolder, settings []byte, members map[string]json.RawMessage) ([]byte, error)`
- `internal/vscode#TestBuildWorkspace`
- `internal/fabricengine#regenerateHubWorkspace`
- `internal/fabricengine_test#TestRegenerateHubWorkspace`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#beamWorkspaceMark`
- `internal/fabricengine#markBeamWorkspace`
- `internal/fabricengine#InstallPairFolderNamer`
- `internal/lymcwd#NamedWorktree`

**Intent:**
Add the general workspace-content seam of D6 and its Added-after-Discussion entry: fabric passes opaque members through, `internal/vscode` owns the workspace keys.

- `WorkspaceContent` is the hook: handed the hub path, the beam slug a workspace file belongs to (empty for the hub's own file) and that worktree's anchor path, it returns extra top-level members as raw JSON keyed by name.
  `InstallWorkspaceContent` stores it process-wide in an `atomic.Pointer`, as `InstallPairFolderNamer` does, and returns a restore function; with none installed a file carries no extra member.
- `workspaceMembers` calls the installed hook; it reads only the names and refuses a member named `beamWorkspaceMark`, which fabric owns:
  `the workspace-content hook returned a member named "lym.generated", the mark fabric owns on a beam's workspace file`.
- `vscode.BuildWorkspace` takes the members and splices each verbatim after `settings`, in name order, each value trimmed of trailing whitespace.
  It refuses a member named `folders` or `settings`, the keys it writes itself: `workspace member "<name>" is a key the workspace builder writes itself`.
  Nil members give today's bytes exactly.
- `regenerateHubWorkspace` passes the members of the hub file (beam `""`, the prime's anchor path) and of each beam's file (the beam's slug and its anchor, from `lymcwd.NamedWorktree`) to `BuildWorkspace`.
  A beam whose anchor `lymcwd.NamedWorktree` cannot read is logged at Warn and its file is built with no hook members, as `beamWorkspaceSettings` degrades on the same failure, so the regeneration still writes it.
  It now builds every file's bytes, the hub's and each beam's, before it writes any, so a refusal from either side is returned with nothing written; a hook refusal is a programmer error, which only a test's hook reaches.
  Write order and the write-only-on-change rule stay as they are.
- `doc.go`'s `# The hub workspace file` section gains the seam: fabric names no workspace key but its own mark and passes the members on unread.

Bound: fabric learns nothing of tasks, verbs or the lym binary, and its existing `internal/vscode` import gains only an opaque member map.

Tests:
- `TestBuildWorkspace` (tier0) gains rows: members spliced after `settings` in name order; nil members give the unchanged bytes; a member named `folders` and one named `settings` each return an error.
- `TestRegenerateHubWorkspace` (tier1) gains rows: with no hook installed the hub file and a beam file carry no `tasks` key;
  a hook returning a member named `lym.generated`, `folders` or `settings` for a beam's file only, and no member for the hub's, makes the regeneration return an error, and neither the hub file nor the beam file changes from its bytes before the call, so a hub file built before the refusal is not written;
  a hook's member is handed the beam's slug and anchor for the beam file and `""` and the prime's anchor for the hub file.
  Each row installs its hook with `InstallWorkspaceContent` and restores it in `t.Cleanup`; the test already serialises its installs of the pair folder namer.

**ImpactSummary:** BuildWorkspace gains a parameter; its only callers are regenerateHubWorkspace's two calls and TestBuildWorkspace, both on this card; regeneration writes nothing until every file builds.

**Commit:** 8: fabric: a workspace-content seam for the workspace files
