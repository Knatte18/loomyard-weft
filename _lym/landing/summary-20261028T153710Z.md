# Render one board file per project, research notes first

The board no longer renders every project, task and note of the hub into one README.md.
README.md is now the root project's file, and every project gets a `project-<slug>.md` beside it.

## What changes in the rendered board

- **One file per project.**
  Every project entry, open or done, renders to `project-<slug>.md`.
  README.md holds the entries whose project is empty; a project file holds the entries whose project is that project.
  Handles, layers, the After cells and the Owner column still read the whole board, so a task waiting on another project's task stands in the same layer as before, and a file's handles have gaps.
- **Projects section.**
  Each file opens its body with `## Projects (n)`, one bullet per open project directly under it: the handle and title linked to its file, then its brief.
  Child projects follow the parent's `order` key, then handle; README.md lists them by handle.
  The lead line "Groups of entries; a beam spawns its members from its own worktree." is gone, and so is the per-project member table with its Standing column, since a project's members now render in full in its own file.
  A done project is a row of its parent file's Done table, with its title linked to its file.
- **Project file header.**
  A project file opens with `# <handle> <title>`, the render time, `Under:` linking up to its parent's file (README.md for a root-level project), the brief, `beam` or `grouping`, and a beam's effective owner.
- **Research first.**
  Notes subsections run `research` first when it is a configured type, then the other types in board.yaml order, then Other.
  The research heading reads `Research`, not `Researchs`.
- **After by handle.**
  The After cell names each open dependency by handle, one per line (`<br>`), in depends_on order.
  A dependency the board holds no handle for, or no longer holds, shows as its slug.

Removed project files are cleaned up by the existing render manifest, so a project that is pruned or renamed leaves no stale file behind.

## Code

- `internal/boardengine/render.go`: `Render` builds README.md and one file per project through `boardView.renderBoardFile`, replacing `renderReadme` and `renderProjects`.
  `boardView` holds the hub-wide facts each file reads: entries by slug and effective owners.
  `notesTypeOrder` and `typeHeading` handle the research ordering and heading.
  `afterCell` replaces the slug list.
  `standingCell`, `projectMembers`, `codeList` and the Member, Kind and Standing columns are removed.
- `internal/boardengine/doc.go`: the `# README` section describes the per-project layout.
- `internal/fabricengine/replay.go`: `isBoardOverview` recognises `project-<slug>.md` at the board root as a rendered overview beside README.md.
  Without it, a board commit touching a project file was not board-only, fell back to a git-level replay and conflicted on board.json; the board re-renders every overview after a replay, so none names an entry change.
  The Bolt paragraph of `internal/fabricengine/doc.go` names the new files.

## Design decisions

- The file prefix `project-` is a renderer constant, not a `board.yaml` key, so no hub base config changes.
- Every project gets a file, done ones included, because prune keeps a done project while a member still names it, and those members need somewhere to render.
- The research-first rule names the `research` type label in the renderer; a hub without that type renders its Notes in board.yaml order as before.
- Design docs keep their `Depends on:` line by slug; only the board files' After cell switched to handles.

## Tests

- `TestRenderReadmeGolden`: the main fixture gains a research note, pinning Research before Bugs and Enhancements, and the After goldens switch to handles.
  The projects row now pins README.md holding only root entries with linked project bullets and a linked done project, the full `project-ship.md` and nested `project-inner.md`, the child order from the `order` key, a multi-handle After cell across projects, and the full set of project files.
  The lone-beam row pins the beam's owner in its own file.
- `TestRenderToDiskManifestCleanup/ProjectRemoved`: a project's file is removed once the project leaves the board.
- `TestSetOwners/CompetingWriteLosesToOrigins` covers the replay of a board commit that touches project files.
- `TestRename_KeepsEntryRewritesDependentsAndReportsMentions` reads the renamed member from its project's file and pins the old project file's removal.
