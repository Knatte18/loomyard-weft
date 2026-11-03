# Webster reads git and card paths at the worktree root, not the anchor

In a hub whose `.lym-anchor` names a subdirectory, Webster treated the anchor as the git worktree.
This blocked every tapestry run in such a hub before it reached code ([#32](https://github.com/Knatte18/loomyard/issues/32), [#31](https://github.com/Knatte18/loomyard/issues/31)):

- `begin-batch` refused before any fork with `gitrepo: open go-git handle at <worktree>/<anchor>: repository does not exist`, because go-git opens only a directory that holds `.git` itself.
- Plan checks joined `root:`-prefixed card paths onto the anchor, so the prefix appeared twice and committed files read as `path-missing` and `glyph-not-found`.

## What changes

`websterengine.Geometry` now carries two roots, each with one meaning:

- `WorktreeRoot` is the git worktree root, the directory holding `.git`, `PATTERN.md` and `go.mod`.
  Every git call opens the repository here, and `planglyph`/`planindex` resolve card paths (`root:`, `//` and plain) against it, as `loom-plan-spec.md` defines.
  `hubgeom.WebsterGeometry` sets it to `l.WorktreePath()`.
- `PaneCwd` is new: the directory Webster's sessions run in, `l.AnchorPath()` in a hub and the target in standalone mode.
  It is used where that directory is meant:
  - resolving transcript-relative write paths, and so as the fork-audit workdir;
  - the working directory of card-verify reruns and the verify gate;
  - rendering `{{.worktree_root}}` and the prompts' plan-directory spelling;
  - reading Merriam's instruction files;
  - locating the session's `_lym` and `.lym`.

`RepoRoot` is removed. It was always equal to the new `WorktreeRoot`, and its one use, the PATTERN directive, now reads `WorktreeRoot`.

Some helpers took one directory for two jobs: resolving a recorded path, and then checking it against the git tree and asking git about it.
`trackedRel`, `suspectBlobs`, `writtenWorktreePaths` and `failBatchInput` now take both roots.
A relative path resolves against `PaneCwd`.
The within-tree check, the repository-relative path and every git probe use `WorktreeRoot`, so the paths they give to `git rev-parse <commit>:<path>` and similar calls are relative to the repository root.

Both roots come from the resolved `lymcwd.Location`, and no Webster code joins or strips the anchor by hand (`PATTERN-cwd-resolution`).
An unanchored hub and standalone mode behave as before, since there the two roots are the same directory.

## Tests

- The `webstercli` verbs fixture already ran at a `backend` anchor but wrote a `.git` link file into the anchor to work around #32.
  The link is gone, so every begin-batch, record-batch, recover-batch, fix-plan and accept-audit test now opens git from a subpath anchor with no `.git`.
- `TestPersistentPreRunE_PlanDirAnchoredAtSubpath` now validates a plan with `root: backend` that edits `backend/README.md`, and asserts no `path-missing` or `glyph-not-found` finding.
- With `hubgeom.WebsterGeometry` set back to the anchor, `TestBeginBatchCmd_HappyPath` fails with #32's error and the validate test fails with #31's `path-missing`.
- The `hubgeom` and `webstercli` wiring tests pin `WorktreeRoot` to the worktree path and `PaneCwd` to the anchor.
  Engine test fixtures that relied on `WorktreeRoot` doubling as the pane's cwd now set `PaneCwd` too.

## Design decisions

- `WorktreeRoot` was redefined to the git root and a separate `PaneCwd` was added.
  Moving the git and plan uses to `RepoRoot` would have avoided the new field, but `WorktreeRoot` would then keep meaning the anchor in a hub.
  The redefinition matches `reedengine.Geometry`, whose `WorktreeRoot` and `PaneCwd` have the same meanings.
- Gate runs (card-verify reruns and the verify gate) keep running in the pane's cwd, the anchor, as before: the hub's gate commands and `lym` itself are run from there.
- The tackler's hand-back on #31 proposed a fix for a `#` the card did not contain. It is out of scope, as the record says.
