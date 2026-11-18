# Card 6 — map-run-writes-index

**Create:**
- `plan:internal/testreach#WriteIndex` -> `func WriteIndex(module Module, sharedDir string) error`

**Edit:**
- `internal/testreach_test#TestReaderIndex`
- `internal/gatecli#runTestMap`
- `internal/gatecli#TestGateTestMap`
- `internal/gatecli#TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`
- `cmd/lym#TestCoverageMapReaderBoundary`

**Extends:**
- `internal/testreach/doc.go`
- `internal/gatecli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/testreach#Module`
- `internal/testreach#Reader.TestsReaching`
- `internal/testcover#WriteMap`
- `internal/state#WriteJSON`
- `internal/gatecli#coverageMapsDir`
- `internal/gatecli#readTestMap`

**Intent:**
Add the writer of D1's index of the shared maps, and make `lym gate test --map` run from the prime its only caller.
The card follows card 5, whose gatecli code first calls `testreach.Open`, so `WriteIndex` arrives with its one caller.

`WriteIndex(module, sharedDir)` builds the reverse table from the shared maps, as `Open` builds it, and writes it through `internal/state` to the index file beside the maps,
with the stamp `Open` checks (card 2): the set of map files in the directory, each with its size and modification time.
Bound: a map rewritten at the same size within one modification-time tick keeps its stamp, so the index can serve its old pairs until the next map run writes it; every answer is advisory.

In `runTestMap`, after `run.mapPackages` returns without error and before the output envelope:
when the call runs in the prime (its maps directory is the hub's shared one, as `coverageMapsDir` picks it) and HEAD did not move (`run.headMoved` is false),
call `testreach.WriteIndex` with the run's module (`moduleDir`, `moduleRel`, `modulePath`) and its maps directory.
- A task worktree's call writes no index: its maps are few and the reader reads them whole.
- A call whose HEAD moved ends its walk early and writes no index, so readers build the table in memory until the next map run (D1's second bound).
- A failed index write is logged at Warn with the path and cause and changes neither the output nor the exit code; the index is a cache, and its absence costs time and refuses nothing (`PATTERN-lym-files`).

`TestCoverageMapReaderBoundary` (card 3) gains the index clause: it fails when a non-test file other than `internal/gatecli/testmap.go` calls `testreach.WriteIndex`,
and it pins that file for the call, so the pinned file failing to hold it fails as the guard's other pins do.

Tests:
- `TestReaderIndex` (tier0) gains the index states `WriteIndex` produces, over map directories it writes with `testcover.WriteMap` under `t.TempDir()`:
  a current index, an index whose stamp differs after a map was rewritten, and an index written before a map file was added.
  The rewritten map changes a reach pair the index holds, and the row moves the file's modification time with `os.Chtimes`, so a reader that trusted the stale index would answer otherwise.
  Each row's `TestsReaching` answers equal those of the no-index row, and `Open` leaves the index file's bytes as it found them.
- `TestGateTestMap` (tier1) gains an assertion on a prime row that measured its packages: the index file exists beside the maps after the call.
  Its row whose HEAD moved mid-walk gains an assertion that no index file exists after the call.
  It gains a prime row whose index path is held by a directory, so the write fails: the call's output and exit code equal those of a passing call, and no index file is written.
- `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` (tier1) gains an assertion that no index file exists under the worktree's anchor maps nor beside the shared maps after the task worktree's call.

Docs: `internal/testreach/doc.go` gains the index's writer and its bound.
`internal/gatecli/doc.go`'s map-run paragraph gains the index write and its conditions; `docs/overview.md`'s `lym gate test --map` sentence gains the same, in one clause.

**ImpactSummary:** The reader's index gains its one writer, and the prime's map run gains one advisory step at its end, logged on failure; two existing tier1 map tests gain an assertion each, the reader-boundary guard gains one clause, and two doc passages gain a clause.
