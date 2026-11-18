# Card 3 — map-run-writes-index

**Edit:**
- `internal/gatecli#runTestMap`
- `internal/gatecli#TestGateTestMap`
- `internal/gatecli#TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`

**Extends:**
- `internal/gatecli/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/testreach#WriteIndex`
- `plan:internal/testreach#Module`
- `internal/gatecli#coverageMapsDir`
- `internal/gatecli#readTestMap`

**Intent:**
Make `lym gate test --map` run from the prime write the reverse index of D1 after its maps, and no other caller.

In `runTestMap`, after `run.mapPackages` returns without error and before the output envelope:
when the call runs in the prime (its maps directory is the hub's shared one, as `coverageMapsDir` picks it) and HEAD did not move (`run.headMoved` is false),
call `testreach.WriteIndex` with the run's module (`moduleDir`, `moduleRel`, `modulePath`) and its maps directory.
- A task worktree's call writes no index: its maps are few and the reader reads them whole.
- A call whose HEAD moved ends its walk early and writes no index, so readers build the table in memory until the next map run (D1's second bound).
- A failed index write is logged at Warn with the path and cause and changes neither the output nor the exit code; the index is a cache, and its absence costs time and refuses nothing (`PATTERN-lym-files`).

`TestGateTestMap` (tier1) gains an assertion on a prime row that measured its packages: the index file exists beside the maps after the call.
Its row whose HEAD moved mid-walk gains an assertion that no index file exists after the call.
It gains a prime row whose index path is held by a directory, so the write fails: the call's output and exit code equal those of a passing call, and no index file is written.
`TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` (tier1) gains an assertion that no index file exists under the worktree's anchor maps nor beside the shared maps after the task worktree's call.

`internal/gatecli/doc.go`'s map-run paragraph gains the index write and its conditions; `docs/overview.md`'s `lym gate test --map` sentence gains the same, in one clause.

**ImpactSummary:** One added step at the end of the prime's map run, advisory and logged on failure; two existing tier1 map tests gain an assertion each, and two doc passages gain a clause.
