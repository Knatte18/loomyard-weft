# lym gate testmap builds in any worktree, with maps kept in that worktree

`testmap-any-worktree` · Task · enhancement, gate, testkit · running · Seed-Child

## Seen (operator, 2026-11-16)

`lym gate testmap` refuses every worktree but the hub's prime, builds at the prime's HEAD, and writes the maps to the hub's board-level `.lym/testcover/` (`testcover.coveragemap.go`, `lymdirs.TestCoverDirName`).
The prime moves under it: other runs land while it builds, and the build ends once HEAD moves.
So `redundancy-candidates-survey` could not build in its own worktree, and its timing and maps would depend on what landed meanwhile.

## Change

- `lym gate testmap` runs in any worktree of the hub, a task worktree included, and builds at that worktree's HEAD, which only its own session moves.
- A run that builds maps for itself writes them under its own worktree, in `.lym/` or `.scratch/`, so they are thrown away with the worktree and never touch the hub's shared maps.
  `lym gate redundant` reads the maps from the same place, so a run's survey reads the maps that run built.
- The hub's shared maps under the board-level `.lym/testcover/` stay as they are for a call from the prime;
  decide in Discussion whether a call from the prime still writes there, or whether every call writes in its own worktree.
- The rest of the build (packages, side-by-side runs, the build lock per package) stays as `testmap-on-command` left it; the lock is per worktree when the maps are.
