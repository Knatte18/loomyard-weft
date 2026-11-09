```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each changed symbol, output or file shape, I list what reads, asserts or documents it and whether the diff covers it.

**Set per change**
- `cmd/testtiming/spawnscan.go` and `keep.go` moved to `internal/testcover`, with their tests:
  - Covered: the `cmd/lym/tierpurity_test.go` allowlist entry is re-pointed at `internal/testcover/spawnscan_test.go`.
  - Covered: the only remaining references are `testcover.ScanSpawns` and `testcover.ScanKeeps` at `cmd/testtiming/redundancy.go:226` and `:315`, so no stale reference to the old copies is left.
  - Covered: `cmd/lym/spawnobservability_test.go` still lists `cmd/testtiming/redundancy.go` and `main.go`, which still run `go list` and `go test -json`. The new `internal/testcover/runner.go` spawns and imports `internal/logger`, so the guard needs no allowlist entry.
- New package `internal/testcover`:
  - Covered: `docs/overview.md` carries the new package line.
  - Covered: the package is a normal non-leaf package, and no PATTERN entry's closed import set names it.
- New `lymdirs.TestCoverDirName`:
  - Covered: `lymdirs` is the single declarer, and `coveragemap.go` takes the directory as a parameter, so the engine names no `.lym` path itself.
  - Covered: its only user is a future verb, which is the intended scope.
- Report shape in `cmd/testtiming/redundancy.go`: the table gains a fourth column, "Contained module-wide".
  - Covered: `renderPackage` and `renderReport` are asserted by `redundancy_test.go`.
  - Not covered: the existing report and the documents that describe it. See F1 and F2.
- `PATTERN-test-speed.md` test economy section, and "Declared PATTERN changes landed": the directive declares no PATTERN changes, so nothing is owed. Commit message and trailers: no `Pattern-Departure:` trailer, and none is needed.

**Findings**

- **F1 — LOW, consistency** — `docs/benchmarks/test-redundancy.md:3-6`: the report's header text is not rewritten by a `-pkg` splice. `spliceReport` keeps the old header, which says a candidate has "every covered block also covered by another test of its package". That is the old module-wide wording. The first `-pkg` run then yields a report with a three-column header and some four-column sections. The writer's summary admits the old sections stay, but not the stale header. Fix: either regenerate the report in this diff, or make the splice refresh the header on a splice run. A third option is to accept the mixed report and say so in the summary.
- **F2 — LOW, consistency** — `pattern/PATTERN-test-speed.md:247-265`: the test-economy section still describes the removed semantics.
  - It says a candidate is judged without package scoping and without the module-wide column.
  - It says the "no coverage" list holds a test that "covers nothing". Under the package view, a test that covers only other packages' blocks is now listed as "no blocks".
  - Fix: add one sentence saying candidates are judged over the package's own blocks, with a "Contained module-wide" column. Widen the "covers nothing" bullet to "covers nothing of its own package".

No other changed symbol or file shape has an unhandled reader, caller, doc or fixture.

**Tool calls (2)**
1. A grep for `cmd/testtiming` references, `impactset` importers and PATTERN mentions, plus the head of `cmd/testtiming/main.go`. My context did not hold the cross-repo references.
2. The `spawnobservability` allowlist, the head of `docs/benchmarks/test-redundancy.md`, and a grep for stale `ScanSpawns`, `scanKeeps` and `keep.go` references outside `internal/testcover`.
