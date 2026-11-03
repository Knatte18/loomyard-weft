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

## From issue #31

Imported from [issue #31](https://github.com/Knatte18/loomyard/issues/31).

## What happened

Run `commonlib-fluid-report-selection` (tapestry, Models hub, anchor `src/csharp/NORCE.Models`) failed at Webster with two blocking findings on a file that exists and is committed:

```
path-missing/2-mapper-report-choice: card 2 path "src/csharp/NORCE.Models/NORCE.Import.CommonLibHarness/README.md#" does not exist on disk and is not a Create target or Rename destination of any card
glyph-not-found/2-mapper-report-choice: target "src/csharp/NORCE.Models/NORCE.Import.CommonLibHarness/README.md#"'s unit does not exist
```

The plan overview had `root: src/csharp/NORCE.Models` and `language: csharp`; card 2 listed `NORCE.Import.CommonLibHarness/README.md` under Edit.
The tackler misread it as a stray `#` in the card and asked the parent to remove it; the card held no `#`.

## Cause

`loom-plan-spec.md` ("Card path resolution") and `contracts/stencils/loom/loom-block-plan-overview.md` define `root:` as worktree-relative and `//` as worktree-root-relative.
In a subpath-anchored hub, `hubgeom.WebsterGeometry` sets `WorktreeRoot` to the anchor path (`internal/hubgeom/webstergeom.go:31`), so `path-missing` (`validate.go` `pathExistsOnDisk`) and `planglyph.Resolve` check every path-shaped ref relative to the anchor.
`normalizeCardPath` joins `root:` onto the card path (`internal/planparser/normalize.go:42-53`), so the file is looked up at `<anchor>/src/csharp/NORCE.Models/NORCE.Import.CommonLibHarness/README.md`, a doubled prefix.
This hits any plain path, `.cs` included, whenever `root:` is set to the anchor as the spec invites.
The csharp test at `internal/planparser/validate_test.go:1507` uses a slash-free `README.md#`, which never reaches the disk check, and no test pairs a non-`.` anchor with a `root:` plan.

## Expected

Either the spec and stencil say `root:` and `//` are anchor-relative (and the plan writer stops setting `root:` to the anchor), or Webster checks against the worktree path consistently.
A test pairing a subpath anchor with a `root:` plan would pin the choice.
The tackler's hand-back should also not name a fix for text the card does not contain.

## Workaround

Spell the entry as an anchor-relative file self glyph, `NORCE.Import.CommonLibHarness/README.md#`, which is never `root:`-joined, then `lym webster rebaseline --card 02` and resume.