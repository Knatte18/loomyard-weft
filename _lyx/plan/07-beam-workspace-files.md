# Card 7 — beam-workspace-files

**Edit:**
- `internal/fabricengine#regenerateHubWorkspace`
- `internal/fabricengine#hubWorkspaceFolders`
- `internal/fabricengine#writeHubWorkspace` -> `func writeHubWorkspace(l *lyxcwd.Location, path string, content []byte) (bool, error)`
- `internal/fabricengine#RegenerateHubWorkspace`
- `internal/fabricengine_test#TestRegenerateHubWorkspace`
- `internal/fabricengine#TestWriteHubWorkspace`
- `internal/fabricengine#TestWriteHubWorkspace_ReadErrorFailsWithReadCause`
- `internal/fabricengine#TestWriteHubWorkspace_RefusesEscapingSymlink`
- `PATTERN.md`
- `pattern/PATTERN-cwd-resolution.md`
- `internal/fabricengine/doc.go`

**Create:**
- `plan:internal/fabricengine#BeamWorkspacePath` -> `func BeamWorkspacePath(l *lyxcwd.Location, beam string) string`
- `plan:internal/fabricengine#beamWorkspaceMark` -> `const beamWorkspaceMark = "lyx.generated"`

**Uses:**
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#MarkBeam`
- `internal/fabricengine#AddOptions`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`

**Intent:**
D12: `RegenerateHubWorkspace` itself changes, so every caller (the fabric verbs and the prime spawn) writes the beam files too, all from the origin records and never from the board.
Each listed pair's origin record is read with `ReadOriginFor`; its parent is its `ParentWorktree`, the prime when that names the prime or is empty, and a pair whose parent worktree is gone or unrecorded counts as the prime's.
A pair whose record cannot be read, such as a records worktree missing its lock directory, is listed in the hub file as an unrecorded pair with a debug log, as `hubWorkspaceFolders` skips an unresolvable pair today, so one broken pair never fails the regeneration or drops the beam files.
The package doc's `RegenerateHubWorkspace` paragraph, which calls the hub file the one workspace file of a hub, is rewritten to name the beam files beside it.
The hub's `<repo>.code-workspace` lists the prime, Bolt, and every pair whose parent is the prime, beams included, and no pair under a beam.
For each pair whose record carries `Beam`, `BeamWorkspacePath` names `<hub>/<repo>-<beam>.code-workspace`, beside the hub file, listing the beam's worktree, its parent's worktree, Bolt, and every pair whose parent is the beam; a sub-beam appears there by its own worktree only and gets a file of its own.
Each beam file carries the mark: a top-level `beamWorkspaceMark` key set to `true` beside `folders` and `settings` (VS Code ignores unknown keys).
After writing, the regeneration lists `<hub>/<repo>-*.code-workspace` and removes, through the hub `os.Root` (PATTERN-fabric-git hub writes), a file only when it parses as JSON carrying the mark and no live beam pair of that slug remains, so a landed beam's file goes with its teardown and an operator's own unmarked file of that name stays.
`writeHubWorkspace` takes the target path, so one writer serves both files; its tests pass `HubWorkspacePath(l)`.
PATTERN-cwd-resolution's entry and its background's Hub containment section name the hub's workspace files, the beam files included: hub-level, written directly in the hub dir with hub-inward folders only, never into a worktree.
The entry's whole overview line becomes:

```
- `PATTERN-cwd-resolution` — Resolving a cwd or worktree root: only `internal/lyxcwd`; an engine is handed absolute paths and never imports it; hub-level containers and the hub's workspace files stay reachable from the hub only. — [background](pattern/PATTERN-cwd-resolution.md)
```

`TestRegenerateHubWorkspace` gains rows: a beam pair `b` (added with `AddOptions.Beam`) with a member `m` (added with `Parent: "b"`) leaves the hub file listing the prime, Bolt and `b` but not `m`, and `<repo>-b.code-workspace` listing `b`, the prime as its parent, Bolt and `m`; a pair whose parent worktree is removed by hand is listed in the hub file, and so is a pair whose origin record is deleted by hand (unrecorded) and one whose record names an empty `ParentWorktree`;
a sub-beam `c` (added with `Parent: "b"` and `Beam: true`) with its own member `n` appears in `b`'s file by its own worktree only, not `n`, and gets `<repo>-c.code-workspace` listing `c`, `b` as its parent, Bolt and `n`;
a pair whose records worktree lacks its lock directory is listed in the hub file and the regeneration succeeds; after `b`'s pair is removed its marked file is gone, while an unmarked `<repo>-b.code-workspace` written by the test stays; a beam made by `MarkBeam` over an existing pair gets its file on the next regeneration.

**ImpactSummary:** The hub workspace regeneration now writes and prunes per-beam files; its callers keep their signature, and writeHubWorkspace's only callers are regenerateHubWorkspace and its three tests on this card.
