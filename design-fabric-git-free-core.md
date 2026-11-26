# Restructure fabricengine into a git-free core so its tests need few repositories

`fabric-git-free-core` · Task · enhancement, fabric, testkit · starting · Open-Window

## Why

`internal/fabricengine` sets the length of most tier1 gate runs: 211 s of a 250 s tier1 run on 2026-11-26, the next package 64 s.
Its tests make about 340 hub copies, about 230 pairs with a push, 35 one-repository hubs and 16 second clones per run.
Trimming and folding tests has been tried several times (`docs/research/speed/fabric-test-speed-survey-2026-11-03.md` and the landings after it); it does not solve it.
The cause is the structure: nearly every rule can only be tested against real git repositories.

## Operator decision (2026-11-26)

Restructure `fabricengine` wholesale so its tests need few real repositories; do not trim the test suite again.

## Direction to discuss

- A git-free core: it reads a state snapshot (refs, worktrees, records, origins) as plain data and decides what to do, returning a plan.
  Its rules are tested in memory over made-up states, with no clone.
- A thin git side: it reads the snapshot from git and carries out the core's plan; only it needs real repositories.
- A few end-to-end tests that pin the seam between the two, in place of hundreds of integration tests.
- How the existing tests move over: which become core tests over states, which stay as git-side tests, which go.
- The test imports that pull `fabricengine` into the gate scope of other engines' changes, and how the split cuts them.

Discussion settles the boundary, the order of the move and the split into tasks for raddle.
