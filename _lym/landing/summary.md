# Tell agents never to build or check out under .scratch, .lym or _lym

A task agent built an external .NET harness in a scratch checkout under `.scratch`.
`.scratch` is a link into the weft, so MSBuild saw each project under two paths and the build failed on a missing project reference.
Agents have also checked out a worktree or clone under `.scratch/`, which nests one git repository inside another.

The instructions lym ships now say that `.scratch`, `.lym` and `_lym` are scratch and record folders, never a build location.
An agent never builds, restores or compiles anything under them, and never creates a git worktree, clone or checkout there.

Closes #140

## What changes

- The Claude and Codex edit-directive stencils (`contracts/stencils/engine/{claude,codex}/edit-directive.md`) carry the ban.
  It replaces the earlier advice to build under `.scratch/` and name the working tree's files by absolute path.
- The orch role contract (`contracts/orchrole/orch-role.md`) carries the same ban in its notes and orch rules.
- The `internal/editdirective` package doc matches the new stencil content.

## Design decisions

- `.scratch` stays a link; it does not become a real directory, per the operator's decision on the issue.
  The fix is instruction text only, with no guard or code change that enforces the ban.
- The absolute-path workaround is dropped rather than kept beside the ban, so the stencils no longer suggest that building under `.scratch` is possible.

## To check by hand

- The stencil prose was not run against a live agent session, so whether agents now follow the ban is unverified.
- Code-review finding `edit-directive-test-pins-stencil-prose` in `internal/editdirective`:
  a test assertion pinned the stencil's wording.
  The latest commit drops that assertion; confirm no remaining test pins the new prose.
