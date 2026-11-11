## Goal

lym gate testmap plans each map at the prime's HEAD, keeps what still holds, and checks what it keeps

Takes: testmap-selector-upkeep-rules' selector with a commit target and its environment stamp; today's lym gate testmap (runTestMap, mapPackages, stale, mapPackage, writeMap) in internal/gatecli, with its blocking slot take and one process per test
Delivers: the plan at H with the Current, Affected and Miss verdicts, re-stamps, marker updates and queued gaps, under the build lock with held locks skipped (D1); outcome entries, spawn verdicts and nothing recorded for an ended run (D4); the kept map with a failure field, and no requeue at the failed HEAD (D5); test_map_max_age_days retired from gateslot, the self-check on every build and one audit round per pass (D7); testcover.ExitRecord's planned HEAD, queued count, verdict counts with miss reasons, mismatch counts and per-package build and run seconds, and the verb's report of them (D12); the redundancy step counting only passing entries as covers and judging a map with a failure field by its entries
Exports: the plan and queue a pass works, the job that measures a queued test list, testcover.ExitRecord's new fields

## Scope

The part changes these packages:

- internal/gatecli (about 850 changed lines)
- internal/testcover (about 350 changed lines)
- internal/gateslot (about 30 changed lines)

## Decisions

### D1 — Staleness is the selector's verdict at the prime's HEAD

A map keeps one `Commit`.
An upkeep pass decides what to measure by calling `coverage-test-select`'s selector with the map and the prime's HEAD `H` as target,
the call the gate makes with the working tree as target.
The verdict per map is one of three:

- **Current**: no affected test and no miss.
  The pass re-stamps the map: `Commit` becomes `H`, the recorded file hashes become `H`'s,
  and every entry is kept but an outcome entry D4 drops, which is dropped and queued.
- **Affected tests**: the pass keeps every unaffected entry but an outcome entry D4 drops, drops the affected ones,
  writes the map at `H` at once, and queues the dropped tests for measurement.
- **Miss**: the pass leaves the map as it is and queues a whole rebuild of the package.
  A whole rebuild's first written round starts from an empty map:
  it holds that round's results and a fresh spawn scan's verdicts (D4), so the map written at `H` holds only entries measured or scanned at `H`,
  and the tests not yet measured are gaps.

Under every verdict but Miss the plan also queues each declared mapped test of the package that has no entry;
a spawn-marked test has one (D4).
So a gap left by an ended pass, a discarded round (D10) or a dropped outcome entry (D4)
is queued again at the next plan, and a Current map with such a gap is re-stamped and has those tests queued.
The plan itself builds nothing and runs no test.

Every write that moves a map's `Commit`, a re-stamp included, sets the format marker's `Commit` and `BuiltAt`
to the map's own, and `BuiltAt` to the time of the write;
it also sets the marker's environment stamp (D6) to the current one.
This plan step reads file content at the map's `Commit` and at `H` through `fabricengine.Fabric.CodeFileAt` and holds no gate slot.
It rewrites a map only under the package's build lock, through `testcover.WriteMap`,
and skips a map whose lock is held, planning it again once the lock frees, under D9's wait rule;
a map still skipped when the pass ends counts as queued work in the exit record (D8), so the next pass plans it again.

So a change in a linked-only package leaves a map current exactly when the selector finds no affected test and no miss.
Until the build of this decision lands, testmap's present rule stands.

Rationale: a carried entry is wrong only where the selector would already wrongly skip that test in a gate,
so upkeep adds no new way to be wrong, and both errors meet the same backstop, the `finalize` landing gate at scope `all`.
A map with gaps is valid: `gate-scope-coverage`'s D3 runs every test its map does not record.
Bound: an entry the selector wrongly keeps lives until its inputs change or D7's audit reaches it.

### D4 — What a measured test records, and which tests are measured

- A passing run records the test's coverage and the commit and time it was measured.
- A run that fails, times out, skips or covers nothing records an outcome entry: that outcome, the commit and the time, and no coverage.
  The plan keeps an outcome entry through a Current verdict and drops it on a miss
  or when any of the package's own files differs between the map's `Commit` and `H`;
  dropping an outcome entry queues that test.
- A run or build ended by D9's yield, D11's stop or a terminating signal records nothing: no outcome entry and no failure.
- A test the spawn scan (`testcover.ScanSpawns`) marks out-of-process or unresolved is recorded with that verdict, is never run,
  and counts as an entry for D1's queue, so it is never queued.
- The redundancy step counts only passing entries as covers, so an unmeasured spawn-marked test is no longer a cover.

Rationale: a spawn-marked test's in-process blocks serve no gate, and these tests are among the costliest to run alone.
Fewer covers can only turn a redundancy verdict from contained to unique, and that step is advisory.
A test with an outcome entry runs in every scoped gate, so keeping the entry across linked changes costs gate time, never a missed test.

### D5 — A failed build keeps the old map

When the coverage build of a package fails at `H`, testmap keeps the package's map at its `Commit` with its entries,
leaves its marker as it is, and records the failure and `H` in a field of its own beside the entries.
A package with no map, or whose map holds no test entry, gets today's failure-only map with `BuildFailure` set,
which the selector and the redundancy step keep reading as they do today.
A package's recorded failure is that field, or a failure-only map's `BuildFailure` with its `Commit`;
a pass does not queue the package again while its recorded failure names the current HEAD.
The redundancy step judges a map with entries and a failure field by its entries.

Rationale: the old map still selects soundly for any tree descended from its `Commit`, so a broken commit costs no precision.

### D7 — The age limit goes; a self-check and an audit replace it

- `test_map_max_age_days` is retired.
- Whenever upkeep builds a package at `H`,
  every kept entry's own-package block keys must appear in the fresh build's block table,
  and its linked functions in the fresh build's function set of each linked package.
  A kept entry that fails this is re-measured in the same job and counted as a self-check mismatch.
- Every pass ends with one audit round once its queue is empty:
  if it can take a slot by D9's rule at once, it re-measures a fixed small number of passing entries, oldest measurement first,
  and a fresh result that differs from the kept one replaces it and is counted as an audit mismatch.
  The number is a named constant in code, not config.

Rationale: D1's rule bounds dependency drift exactly, D6 bounds the environment, and the audit measures what no rule models.
A mismatch count is evidence of a selector gap or a nondeterministic test, where a weekly rebuild hid both.
Bound: the audit runs once per pass, so on a day with no landing it re-measures nothing; with no landing, no entry goes newly wrong.

### D12 — Upkeep measures in the prime worktree and reports its work

- Upkeep stays in the prime worktree, as testmap does today, with `testmap-cross-package`'s two fresh dirty reads over every linked package.
- The exit record, `testcover.ExitRecord`, replaces `Unbuilt` with:
  the planned HEAD, the queued package count, which counts the queued work left when the pass ended,
  the plan's counts of current maps, maps with affected tests and misses by reason,
  the self-check and audit mismatch counts,
  and per measured package its build seconds, run seconds and test count.
- The verb's report prints the same.

Rationale: the counts are the live form of the measurements the report asks for:
how often a landing misses rather than rolls, and how a job's time splits between build and runs.

### Added after Discussion (parent, 2026-11-11): testmap-cross-package is held until this record is approved

Decision: The run of testmap-cross-package is paused at Run-Shed, so its Darn-Write writer writes no code. It stays held until this decision record is approved and D13's changes for testmap-cross-package are carried onto its stored record.

Rationale: The operator decided in chat on 2026-11-11 to hold testmap-cross-package, coverage-test-select and gate-scope-coverage until this design is decided; the orch paused the run with lym batten pause. The board shows the old running status because the pause leaves the entry's status.

## Constraints

- PATTERN-gate-slot: every coverage build and test run of upkeep holds a slot; the plan step builds and runs nothing, so it holds none.
- PATTERN-config-layers: `test_map_max_age_days` (gateslot, a strict loader), `test_map_batch` and `test_map_every` (dobbyengine)
  leave their templates as retired keys of their modules, so `hubreconcile.Ensure`'s walk removes them from a hub base file
  and a strict load does not refuse a hub that tuned one; `test_map_every`'s 0 is carried by D8's migration.
- PATTERN-fabric-boundary: file content at a map's `Commit` and at `H` is read only through `fabricengine.Fabric.CodeFileAt`.
- PATTERN-tolerant-records: D4's entry fields, D5's failure field, D6's stamp and D12's exit record fields are added fields;
  a read refuses no unknown key, and the retired landing count stays readable in an old dobby state record.
- PATTERN-child-processes: a `--upkeep` pass stays registered as `runproc.RoleTestMap`, and a pass's test processes end through its own context.
- PATTERN-daemon-wakeups: D9's wait for a slot.

## Auto-mode assumptions

- No measurement runs before this design: each changes cost, not correctness, and D12 makes upkeep report them as it runs.
- Spawn-marked tests are never measured, hand calls included.

## Open risks

- The non-body miss stays the largest cost: the Sketch counts one in at least 95 of 118 weekly code commits,
  and each one rebuilds every map that links the changed file.
- A selector bug is shared by the gate and upkeep; D7's self-check and audit counts are its only detectors before a landing gate fails.

## Acceptance criteria

- D1: after a landing that changes the body of a linked function no test of B's map covered, the plan re-stamps B's map to HEAD
  with every entry kept and the marker's `Commit` equal to the map's, and builds and runs nothing; the selector then reads it as no miss.
- D1: after a landing that changes the body of a linked function test T covered, the plan writes B's map at HEAD without T's entry
  and with every other entry, and the job then measures T alone.
- D1: after a landing with a non-body change to a linked file, the plan leaves B's map as it was and queues a whole rebuild of B;
  after the rebuild's first round the map at HEAD holds no entry measured before HEAD.
- D4: a test that fails alone gets an outcome entry; a Current verdict keeps it, and a change to one of B's own files drops it.
- D4: a test the spawn scan marks out-of-process is recorded with its verdict, is never run and leaves the queue empty.
- D4: the redundancy step counts no outcome entry and no spawn-marked test as a cover.
- D5: a coverage build failure at HEAD keeps the package's earlier entries, `Commit` and marker, records the failure with HEAD,
  and the next pass at that HEAD queues nothing for the package; the redundancy step still judges by the kept entries.
- D7: a hub base file holding `test_map_max_age_days` loads after the reconcile walk; no map is rebuilt for its age.
- D7: a kept entry whose block key is absent from a fresh build's block table is re-measured and counted as a self-check mismatch.
- D7: a pass whose plan finds every map current still runs one audit round when it can take a slot,
  and counts a differing result as an audit mismatch.
- D12: the exit record names the planned HEAD, the queued count, the plan's outcome counts with miss reasons and the mismatch counts,
  and the dobby reads it.
