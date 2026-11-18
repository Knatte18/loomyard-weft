# Worktree .scratch is a symlink into .lym/scratch; tools given a path through it see each project under two paths

`scratch-symlink-two-paths` · Note · bug, fabric, cwd

Imported from [issue #140](https://github.com/Knatte18/loomyard/issues/140).

A task agent needed a scratch checkout to build an external .NET harness. The worktree's `.scratch` directory is a symlink into the weft's `.lym/scratch`.
The agent passed a repository path through that symlink to the build, and MSBuild saw each project under two paths, the symlinked one and the resolved one.
The build then failed on a missing project reference; only the fully resolved real path built.

Abnormal: the instructions send scratch files to `.scratch/`, yet a path under it is not a safe build input, and nothing says so.
The agent lost build attempts to the duplicate-path failure.

To do: document in the agent instructions that tools which compare or deduplicate paths (MSBuild, git, other build systems) need the resolved real path of a `.scratch` location, or have lym expose the resolved path to agents, or make `.scratch` a real directory.

Provenance: task td-stick-slip-static-offset; note Darn-Write.md.