# Webster begin-batch opens go-git at the anchor, failing in a subpath-anchored hub

`webster-subpath-anchor` · Note · bug, webster, cwd

Imported from [issue #32](https://github.com/Knatte18/loomyard/issues/32).

## What happened

Run `commonlib-fluid-report-selection` (tapestry, Models hub, anchor `src/csharp/NORCE.Models`) blocks at Webster Batch 01 before any fork, on every re-launch:

```
Batch 01: begin-batch refused before any fork: websterengine: head sha ...: gitrepo: open go-git handle at <worktree>/src/csharp/NORCE.Models: repository does not exist
```

The anchor is a subdirectory of the git worktree; `.git` sits at the worktree root, not in the anchor.
websterengine's head-sha lookup opens the go-git handle at the anchor path, so go-git finds no repository.

## Expected

The head sha is resolved from the git top level (or the open uses go-git's `PlainOpenWithOptions` with `DetectDotGit: true`), so Webster runs in a subpath-anchored hub.

## Related

Same root as #31: `hubgeom.WebsterGeometry` sets `WorktreeRoot` to the anchor path (`internal/hubgeom/webstergeom.go:31`), and Webster code then treats it as the git worktree.
A test of Webster batches in a hub with a non-`.` anchor would catch both.
This blocks every tapestry run in this hub from reaching code.

## Issue comment, 2026-11-03

Root cause pinned: `internal/gitrepo/gogit.go` `goGit()` opens with `git.PlainOpenWithOptions(r.path, &git.PlainOpenOptions{EnableDotGitCommonDir: true})` and no `DetectDotGit`, so any `gitrepo.Repo` made at a subdirectory of a worktree fails to open; webster's `headSHA` gets the anchor path from `hubgeom.WebsterGeometry`. The git CLI walks up on its own, which is why only go-git reads fail. Minimal fix: add `DetectDotGit: true`, plus a test that opens a `Repo` at a worktree subdirectory and reads `CurrentSHA`. Still blocking run `commonlib-fluid-report-selection` in the Models hub.