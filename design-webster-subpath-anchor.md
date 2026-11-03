# Webster treats the anchor as the git worktree in a subpath-anchored hub

`webster-subpath-anchor` · Task · bug, webster, cwd, parsers

Two Webster faults in a hub whose fabric is anchored in a subdirectory by `.lym-anchor`, from [issue #32](https://github.com/Knatte18/loomyard/issues/32) and [issue #31](https://github.com/Knatte18/loomyard/issues/31).
Both block every tapestry run in the NORCE Models hub (anchor `src/csharp/NORCE.Models`) from reaching code; #32 is urgent (operator, 2026-11-03).

## Shared root

`hubgeom.WebsterGeometry` (`internal/hubgeom/webstergeom.go`) sets `WorktreeRoot` to `l.AnchorPath()`, and Webster code treats `WorktreeRoot` as the git worktree and as the base of worktree-relative plan paths.
In a subpath-anchored hub the anchor is not the worktree: `.git` and the plan's path base sit at the worktree root, which the geometry carries as `RepoRoot`.

## 1. `begin-batch` opens go-git at the anchor (#32)

Webster Batch 01 refuses before any fork, on every re-launch:
"begin-batch refused before any fork: websterengine: head sha ...: gitrepo: open go-git handle at <worktree>/src/csharp/NORCE.Models: repository does not exist".

Expected: every git operation Webster runs (head sha and any other go-git or git call) opens the repository at the worktree root, never at the anchor.

## 2. Plan paths are checked against the anchor (#31)

`loom-plan-spec.md` ("Card path resolution") and `contracts/stencils/loom/loom-block-plan-overview.md` define `root:` as worktree-relative and `//` as worktree-root-relative.
With `WorktreeRoot` set to the anchor, `path-missing` (`validate.go` `pathExistsOnDisk`) and `planglyph.Resolve` look a card path up at `<anchor>/<root:>/<path>`, a doubled prefix, so a committed file reads as missing (`path-missing` and `glyph-not-found` on `src/csharp/NORCE.Models/NORCE.Import.CommonLibHarness/README.md#`).

Expected: Webster resolves `root:`, `//` and plain card paths against the worktree root, as the spec says; the spec and stencil stay as they are.

## Both

- The fix follows `PATTERN-cwd-resolution`: the worktree root and the anchor come from the resolved `lymcwd.Location`, and no Webster code joins or strips the anchor by hand.
  Each Webster use of `WorktreeRoot` is checked and moved to the root it means: git and plan paths to the worktree root, Webster's own state under `_lym` to the anchor where it lives today.
- A test runs Webster's begin-batch and the plan path checks in a hub with a non-`.` anchor and a `root:` plan, so both faults are pinned.
- The tackler's hand-back on #31 named a fix for a `#` the card did not contain; noted here, not in scope.
