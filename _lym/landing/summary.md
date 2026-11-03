# Webster: use the worktree root, not the anchor, in a subpath hub

In a hub whose fabric is anchored in a subdirectory by `.lym-anchor`, Webster treated the anchor as the git worktree.
`begin-batch` then refused before any fork, because go-git was opened at `<worktree>/<anchor>` where no `.git` exists.
Plan validation looked each card path up under `<anchor>/<root:>/<path>`, a doubled prefix, so committed files reported `path-missing` and `glyph-not-found`.
Both faults blocked every tapestry run in such a hub, such as the NORCE Models hub.

Closes #32
Closes #31

## What changes

`hubgeom.WebsterGeometry` now sets `WorktreeRoot` to `l.WorktreePath()`, the directory holding `.git`.
Every git call and every plan-path resolution (`root:`, `//` and plain card paths, in `path-missing` and `planglyph.Resolve`) reads it, so they resolve against the worktree root as `loom-plan-spec.md` and the plan overview stencil already say.
Neither document changes.

`websterengine.Geometry` gains `PaneCwd`, the directory Webster's sessions run in, and drops `RepoRoot`, which `WorktreeRoot` now covers.
Hub mode sets `PaneCwd` to `l.AnchorPath()`; standalone mode sets it to the target, as it does `WorktreeRoot`.
Code that meant the session's directory moved from `WorktreeRoot` to `PaneCwd`:

- the fork audit workdir and the `CheckParent`/`CheckFork` write-policy checks;
- the suspect-path resolution and the `_lym` exclusion in `suspect.go`;
- the card-verify and `lym webster verify` working directory;
- the `{{.worktree_root}}` token and plan directory spelling in the fork, Master and verify-fixer prompts.

Webster's own state under `_lym` stays under the anchor.

## Design decisions

- The roots come from the resolved `lymcwd.Location` (`PATTERN-cwd-resolution`); no Webster code joins or strips the anchor by hand.
- `trackedRel` and `suspectBlobs` take both roots: a relative path is resolved against `PaneCwd`, then tested for membership in the tracked tree of `WorktreeRoot`.
- In an unanchored hub the two roots are equal, so behaviour there does not change.

## Tests

The `webstercli` verbs fixture and the `websterengine` helpers now build a hub with a non-`.` anchor and a `root:` plan.
They pin `begin-batch`, record-batch, the plan path checks, the two-root suspect helpers with a subdirectory pane cwd, and the batcher size source.

## To check

- The batcher's disk-size source takes its worktree-root base from `WorktreeRoot`; the last review left this open as `disksizes-worktree-root-base-unpinned`.
  Confirm every size-source construction in `Run`, `rebaseline` and the batcher uses the worktree root in a subpath-anchored hub.
- Run a real `lym webster` batch in a subpath-anchored hub; the tests use fixtures, not a live session.
- Out of scope: the tackler's hand-back on #31 named a fix for a `#` the card did not contain.
