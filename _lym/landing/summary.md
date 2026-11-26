# Map unresolved tier1 tests for reach, and name each caveat's cause

The reach report and `lym gate reach` listed functions as reached by no mapped test
although a tier1 test called them directly, with a fresh worktree map.

The cause was in the map run.
The spawn scan marks a test `unresolved` when it makes a call the scan cannot follow, such as through a func value.
`mapPackage` entered every spawn-marked test with its verdict alone and never ran it,
so the reader reached nothing through it.

The single caveat line also folded every cause together,
so a reader could not tell a map defect from a real coverage gap.

Closes #250

## What changes

- **Map run:** only a test marked out of process stays unrun.
  A test marked unresolved alone runs, and its entry holds its coverage beside its verdict.
- **Which entries answer reach:** the new `MapTest.HoldsCoverage` includes unresolved entries.
  `lym gate reach`, the reach report and the mutation report's possible killers now count them.
- **Format versions:** `MapFormatVersion` goes to 3.
  The shared index goes to version 2, since it now holds each package's unseen tests.
- **Older maps:** the reach refresh remaps a changed package whose worktree map has an older format.
  Such a map also no longer overrides a later shared map merely because its file hashes match.
- **Caveats:** there is one per cause, naming up to five tests and counting the rest.
  The causes are tests that run code in another process, tests with calls the spawn scan cannot resolve,
  and tests whose run alone failed, timed out or was skipped.
  Tests of the function's own package and of the packages linking it are listed apart.
  Another caveat names the tiers that no map holds, tests with no tier included.

## Design decisions

- Tests marked out of process stay unrun: the map run builds no `lym` for them to spawn.
- `MapTest.Measured` is unchanged, so no judge uses an unresolved test's coverage.
  Such a test may also run module code in another process.
  Its coverage can show that a function is reached, and it still yields a caveat for a function nothing reaches.
- All linking packages share one caveat per cause, with each test named `<package>.<Test>`.
  One set per linking package would let a widely linked function fill the capped note.
- A full `lym gate test --map` now also runs the unresolved tier0 and tier1 tests, so it takes longer.
- Existing shared maps keep their empty unresolved entries until the prime's next map run.
  The reach refresh rebuilds an older worktree map only for a package holding a changed function.
  Elsewhere an older worktree map is still read unless the shared map was built later.

## Check by hand

None of these was run:

- A real `lym gate test --map` over `internal/battencli` and `internal/fabricengine`, followed by `lym gate reach`
  on `hubCancelDeps`, `Topology.SettleGonePairTeardown` and `Topology.RemoveLocal`.
  Each should name its tier1 callers.
- How much longer a full map run takes with the unresolved tests included.
- The new tests against the unfixed code.

Confirm these findings against the diff:

- No text assertion on the reach step's assembly of caveats,
  in `internal/gatecli/reach_integration_test.go`.
  `TestGateTestReachStep` now compares exact caveat lists.
- `MapFormatVersion` written but never read, so an older worktree map with matching file hashes kept the miss,
  at `reachStep.staleLinked` in `internal/gatecli/reach.go`.
  `Map.HasCurrentFormat` now reads it there and in `worktreeMapWins`.
- No caveat for an unseen test of another package that calls the function,
  at `reachStep.caveats` in `internal/gatecli/reach.go`.
  `Reader.UnseenLinkerTests` now supplies those caveats.

## Merge danger

**Door:** two-way
**Blast radius:** gate

The `lym` binaries of the hub's other worktrees read the shared maps and index, which this change writes in a new shape.
An older binary's judges skip unresolved entries through `MapTest.Measured`.
It reads a version 2 index as absent and builds the table in memory.
The affected surfaces are the reach and mutation report steps and `lym gate reach`, all under `lym gate`.
