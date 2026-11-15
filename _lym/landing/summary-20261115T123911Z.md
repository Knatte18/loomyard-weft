# Tell agents that .scratch is a link into the weft worktree

Closes #120.

A task worktree's `.scratch` is a directory link into the pair's weft-side `.lym/scratch`.
A build under it resolves a relative project reference from the link's target, not from the working directory.
A card that asked for a no-git build of a project with relative references under `.scratch/` therefore failed.
The agent worked around it by copying the project elsewhere.

## What changes

- The shared edit directive, `contracts/stencils/edit/edit-directive.md`, which every spawned role's opening stencil carries, now says:
  - `.scratch` links into the task's own weft worktree, so a file under it stays in the role's own worktree;
  - a build under `.scratch/` resolves a relative reference from the link's target;
  - a project built there names the working tree's files by absolute path.
- The `internal/editdirective` package doc lists the new statement among what the directive says.
- The `internal/fabricengine` package doc's scratch-link section names this cost of the link.

## Design decisions

- **The link stays; `.scratch` does not become a real directory.**
  The board entry allowed either fix.
  The link exists so that writing scratch files leaves the `go test` cache keys of source-scan tests unchanged (#771).
  A real directory at the anchor changes its own modification time whenever an agent creates or removes a file under it.
  Every test that opens the module root hashes that time.
  A real directory would turn every scratch write into a cache miss for those packages.
- **The second concern is answered in the directive, not by moving the target.**
  The link's target is the pair's own weft worktree, not another task's.
  The directive now says this, so the rule to work only in your own worktree no longer reads as contradicting the link.
- The Claude and Codex engine copies of the directive, rendered into assistant seats, are unchanged: neither tells a seat to write scratch files.
- No test pins the new sentences; the directive's tests check its rendering, not its wording.

## To check by hand

- Read a freshly rendered darn or webster prompt and confirm its `## Editing files` section carries the three new lines.
