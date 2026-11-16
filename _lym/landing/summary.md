# lym gate testmap builds in any worktree and keeps its maps there

`lym gate testmap` used to refuse every worktree but the hub's prime.
It built at the prime's HEAD and wrote the maps to the hub's board-level `.lym/testcover/`.
Other runs land on the prime while a build runs, and the build ends once HEAD moves.
So a survey run such as `redundancy-candidates-survey` could not build in its own worktree,
and its timing and maps depended on what landed meanwhile.

## What changes

- `lym gate testmap` runs in any worktree of the hub and plans and measures at that worktree's HEAD.
  It still refuses a directory outside every hub, and the refusal now points at any worktree of a hub.
- A new `coverageMapsDir` in `internal/gatecli/testmap.go` picks the maps directory.
  From the prime, told by `hubgeom.IsPrime`, it is the hub's shared `testcover.Dir(BoltDir(hub))`, as before.
  From any other worktree it is `testcover.Dir(location.AnchorPath())`, the worktree's own `.lym/testcover/`.
  Those maps are thrown away with the worktree and never touch the shared maps.
- A package's build lock already lies beside its map, so it is now per worktree whenever the maps are.
  The rest of the build (package planning, side-by-side runs, the slot, the audit round) is unchanged.
- `lym gate redundant` reads its map through the same `coverageMapsDir`,
  so a run's survey reads the maps that run built.
  Its rebuild hint now says to run `lym gate testmap <pkg>` from this worktree.
- `testcover.Dir` takes any root now, a board directory or a worktree anchor, and its doc says which holds which.
  The stale mention of an "exit record" there is gone.
- `hubgeom.PrimeWorktreePath` is deleted: the testmap refusal that named the prime was its last caller.

## Design decisions

- The body left one question for Discussion: should a call from the prime still write the shared maps?
  It does.
  The gate's advisory redundancy report step reads the shared maps from every worktree.
  If the prime wrote into its own worktree too, nothing would feed those maps.
- The redundancy report step in `lym gate test` keeps reading the shared maps.
  The entry asks only that `lym gate redundant` follow the worktree.

## Docs

The `internal/gatecli` and `internal/testcover` package docs, the `TestCoverDirName` doc in `internal/lymdirs`, and the `lym gate testmap` lines of `docs/overview.md` now describe where each worktree's maps live.

## Tests

- `TestGateTestMap`'s refusal table drops the two rows that refused a pair worktree and its records sibling.
  Its outside-every-hub row carries the new way forward.
- New `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` commits a module in a pair's code worktree.
  It runs `lym gate testmap` there and checks three things:
  the package is built, its map lies under the pair's own `.lym` at the pair's HEAD, and the hub's shared maps hold none.
  It then checks that `lym gate redundant` from the pair reads that map.
