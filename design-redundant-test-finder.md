# A mechanical way to find redundant tests: package-scoped coverage, a map per package, a check on a diff

`redundant-test-finder` · Task · sketch, testkit, gate

Decision record: [Sketch](decision-redundant-test-finder.md)

## Seen (orch, 2026-11-08)

`PATTERN-test-speed` ("Test economy") forbids a test that covers no behaviour another test does not, and counts a redundant test as a finding.
Only review enforces it.
Its tool, `go run ./cmd/testtiming -redundancy`, measures coverage over the whole module (`-coverpkg` names every package), so a test's blocks include the fixture packages' blocks and almost every test looks unique: `docs/benchmarks/test-redundancy.md` lists no candidate in `internal/fabricengine`.
The coverage study `docs/research/speed/fabric-test-redundancy-2026-11-08/` scoped coverage to the package under test.
Its scripts (`coverage-runner.sh`, `analyze-coverage.sh`) ran each top-level tier 1 test once on its own, serially, and compared the covered blocks:
370 of 590 tests were contained, and a greedy walk, slowest first, found 329 that can go together without uncovering a block.
The run took about four minutes of standalone test time for that one package.
The study also showed the limit of the method: several contained tests guard archive tags, deletes on origin and user content, which block coverage does not see.

## Goal (operator, 2026-11-08)

A mechanical way to find redundant tests, so they stop piling up, at a cost the gates can carry.
Discussion settles the design and its stages; the parts below are the orch's proposal, to be weighed, cut or reordered.

## Decision after the Motif (operator, 2026-11-08): D6's trigger

This replaces D6's trigger in the decision record; the rest of D6 stands.
The prime parent's dobby does not refresh maps on every prime HEAD move: rebuilding a large package such as `internal/fabricengine` takes minutes and holds a gate slot that runs need.
It refreshes periodically, the way it starts the board tidy: after every `test_map_every` landings, a dobby config key beside `board_tidy_every`, where 0 never refreshes.
A refresh still runs `lym gate testmap --max <test_map_batch>` on the packages stale at that moment, and the leftovers the cap left unbuilt wait for the next refresh.
The `test_map` on/off key is dropped: `test_map_every: 0` turns the refresh off.

## Proposed parts

1. **Tool scope.** `cmd/testtiming -redundancy` scopes coverage to the package under test, as the study did, and reports the contained tests and the greedy removable set per package.
   It runs one package at a time (`-pkg`), so each check is scoped to one module.
   Weigh whether the module-wide view stays as an option, since a cut that survives it is the safer one.
2. **Coverage map per package.** The tool stores, per package, which blocks each top-level test covers, with the commit it was built from.
   A change to the package on `main` marks its map stale; other packages' maps stay valid.
3. **Map maintenance.** One job rebuilds stale maps, one package at a time, outside every gate, for example after landings or on a schedule.
   That job alone pays the serial cost; no run waits on it.
   Open: where the job runs (dobby, a hub command, a darn docs task), and where the maps live (tracked, or machine-local under `.lym`).
4. **Gate check on a diff.** A gate step runs only the tests the diff adds or changes, with coverage, and compares them with the package's map.
   A test whose package-scoped blocks other tests all cover becomes a finding for Code-Review that names the covering tests; it never fails the gate and deletes nothing.
   A test carrying `//testtiming:keep <reason>` is not flagged.
   With a stale map for a file the diff touches, the finding says so rather than reporting nothing.

## Constraints and known limits

- Per-test coverage needs the tests to run one at a time: Go's coverage counters (`runtime/coverage`) are process-wide, so tests under `t.Parallel` mix their counts.
  A test cannot record its own coverage inside the normal parallel tier run, so the map cannot come for free from the gates.
- A per-module redundancy test inside the suite is ruled out: it would cost every run touching a large package minutes, and a test binary that launches test binaries is the pattern behind the 2026-11-08 tmux fork loop.
- Block coverage is evidence, not a verdict: no part refuses or deletes a test on its own.
- Weigh a smaller first stage: part 1 alone, run per module on demand as a darn docs task followed by a cleanup darn, with parts 2 to 4 only if redundant tests keep piling up.
