# Worktree .scratch is a symlink into .lym/scratch; tools given a path through it see each project under two paths

`scratch-symlink-two-paths` · Note · bug, fabric, cwd

Imported from [issue #140](https://github.com/Knatte18/loomyard/issues/140).

A task agent in the cs hub (task td-stick-slip-static-offset, Darn-Write) made a scratch checkout under `.scratch` to build an external .NET harness.
`.scratch` is a link into the weft, so MSBuild saw each project under two paths and the build failed on a missing project reference.

## Decision (operator, 2026-11-18)

Nothing is ever built in `.scratch`, `.lym` or `_lym`: they are scratch and record folders, never a build location.
`.scratch` stays a link; it does not become a real directory.

## To do

Tell agents in the instructions lym ships (stencils and skills) that they never build, restore or compile anything under `.scratch`, `.lym` or `_lym`.
