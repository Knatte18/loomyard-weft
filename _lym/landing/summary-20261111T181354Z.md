# The hub workspace file lists every pair, named with its project's handle

The hub workspace file `<hub>/<RepoName>.code-workspace` used to list the prime, `Bolt` and only the pairs whose parent is the prime.
A pair under a beam appeared only in that beam's own file, so opening the hub file left a beam's tasks out of the window.
Every folder was named by its slug alone, so nothing showed which project a pair belonged to.

## What changes

- `RegenerateHubWorkspace` lists every pair of the hub in the hub file, sorted by slug, the pairs under each beam included at any depth.
  The machine restart launcher opens only this file, so it now reaches every pair.
- A pair's folder in the hub file takes the name the process's installed `PairFolderNamer` gives its slug, and its slug otherwise.
  `fabricengine.InstallPairFolderNamer` installs one, mirroring `InstallWorktreeSeam`.
- `boardengine.WorkspaceFolderNames` names each entry that belongs to a project `<P#>: <slug>`, such as `P1: adopt-worker-record`.
  An entry with no project, or whose project the board does not hold with a handle, keeps its slug.
  An unreadable board logs a Warn and names nothing.
- The lym root pre-run installs `boardengine.WorkspaceFolderNames` as the namer, beside the worktree seam.
  The names are read at every regeneration, so a regeneration after a board change brings them current.
- Beam workspace files are unchanged: the same folders, each named by its slug.

## Design decisions

- **A general seam instead of a board read in fabric.**
  PATTERN-fabric-boundary forbids fabric learning anything new of the board.
  Fabric hands the namer the hub path and gets back names keyed by slug; it knows nothing of what the namer reads.
- **The names are read before the hub workspace lock is taken.**
  That lock is a leaf lock, and the board's read takes the board's swap lock.
  The namer therefore returns names for every entry rather than for the listed slugs, since the listing happens under the lock.
- **"Belongs to a project" is the entry's `project` field.**
  A task in a project nested under a beam takes the nested project's handle, not the beam's.
  The console window title's nearest-beam rule was not reused: the folder names the entry's own project.

## Tests

- `TestRegenerateHubWorkspace` gains a sequential case installing a fake namer.
  It checks the namer gets the hub path, the hub file takes its names, an empty name keeps the slug, and the beam file keeps slugs.
  The beam and sub-beam cases now expect the beams' members in the hub file too.
- `TestWorkspaceFolderNames` in `boardengine` covers entries in a project, a nested project, no project, a missing project, and an unreadable or absent board.
- `TestPersistentPreRunE_InstallsSeams` requires the root pre-run to install the pair folder namer.

## Docs

The `fabricengine` and `boardengine` package docs and `docs/overview.md` describe the new listing and naming.
