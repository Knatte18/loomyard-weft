# lym gate testmap builds in any worktree, with maps kept there

`lym gate testmap` no longer refuses every worktree but the hub's prime.
It runs in any worktree of the hub and builds at that worktree's HEAD, which only its own session moves.
Before, a task could not build maps for itself:
the prime moves as other runs land, and a build ends once HEAD moves, so a survey's timing and maps depended on what landed meanwhile.

## What changes

- From the prime, `lym gate testmap` reads and writes the hub's shared maps under the board-level `.lym/testcover/`, as before.
- From any other worktree, it reads and writes that worktree's own maps under its anchor's `.lym/testcover/`.
  They go with the worktree and never touch the shared maps.
- `lym gate redundant` reads the maps from the same place as `lym gate testmap` from the same worktree writes them,
  so a run's survey reads the maps that run built.
  Its rebuild hint now says to build from this worktree.
- The build itself is unchanged: packages, side-by-side runs, the per-package build lock.
  The lock lies beside the map, so it is per worktree as the maps are.
- The call refuses only outside every hub, and the refusal now says to run from a worktree of a hub.

Closes no issue.

## Design decisions

- A call from the prime still writes the hub's shared maps; every other call writes its own.
  The choice is one function, `coverageMapsDir` in `internal/gatecli/testmap.go`, which both verbs use, so they cannot disagree.
- `testcover.Dir` now takes the directory whose `.lym` holds the maps, a board directory or a worktree anchor, instead of only a board directory.
  `lymdirs.TestCoverDirName` and the docs describe both uses.
- `hubgeom.PrimeWorktreePath` is removed: only the old refusal used it.
- The redundancy report step still reads the hub's shared maps from every worktree.
  Only the two `lym gate` verbs follow the worktree.

## To check

- `pattern/PATTERN-test-speed.md` named the prime as the only place the maps are built, in its coverage-run bullet and in its testmap description.
  Both now say the worktree's HEAD and the two map locations; check that no other prime-only wording remains there.
- Not run by hand: `lym gate testmap` from a real task worktree against a real module.
  The new integration test `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` covers a stand-in build from a task worktree,
  and `lym gate redundant` reading its map.
