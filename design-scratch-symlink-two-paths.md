# Agents must never build or check out a repository under .scratch, .lym or _lym

`scratch-symlink-two-paths` · Task · bug, stencil · starting · Seed-Child

Imported from [issue #140](https://github.com/Knatte18/loomyard/issues/140).

A task agent in the cs hub (task td-stick-slip-static-offset, Darn-Write) made a scratch checkout under `.scratch` to build an external .NET harness.
`.scratch` is a link into the weft, so MSBuild saw each project under two paths and the build failed on a missing project reference.
Agents have done this before: checking out a worktree or clone under `.scratch/` to build something, which puts a git repository inside another git repository.

## Decision (operator, 2026-11-18)

Nothing is ever built in `.scratch`, `.lym` or `_lym`: they are scratch and record folders, never a build location.
No agent ever creates a git worktree, clone or checkout under them.
`.scratch` stays a link; it does not become a real directory.

## To do

Tell agents in the instructions lym ships (stencils and skills) that they never build, restore or compile anything under `.scratch`, `.lym` or `_lym`,
and never create a git worktree, clone or checkout there.
