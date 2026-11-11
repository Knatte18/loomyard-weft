## Goal

Decide how the per-test coverage maps are kept fresh, from the research report
`docs/research/speed/testmap-upkeep-2026-11-11/report.md`,
and what that changes in the tasks and notes built on the maps.

## Scope

In:

- The staleness rule of `lym gate testmap`, its unit of work and what it writes.
- What a map records per test, as far as upkeep needs it.
- When the prime dobby starts upkeep, how upkeep holds and gives back a gate slot, and how it orders its work.
- The config keys upkeep retires or replaces.
- The changes this design asks of the entries the maps serve.

Out:

- Sharing maps across machines: the maps stay machine-local under `testcover.Dir`.
- Narrowing the selector's non-body miss: D13 files it as a note.
- The gate scope itself (`gate-scope-coverage`) beyond the one change D13 names.

This record is a design for the build tasks raddle files from it; it lands no code itself.

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

### D2 — One selector serves the gate and upkeep

`coverage-test-select`'s selector is built once in `internal/testcover` with these rules beyond its D4 and D5:

- It takes its target as an argument: the working tree, or a commit read through `fabricengine.Fabric.CodeFileAt`.
- The package's own non-test Go files are compared by function, as a linked package's are,
  in place of D5's miss for any change to the package's own files.
- The package's own files other than its Go files are every tracked file under its directory tree
  down to the next package directory but the package's own Go files:
  `testdata` is included, Go files under it among them.
  A change to one that `inert_paths` does not exempt, or that a `go:embed` pattern of the package names, is a miss.
- A test file is compared with every top-level `Test` function's text removed.
  A test whose `Test` function text changed, or that is new, is affected;
  any other change to a test file, such as a helper, `TestMain`, an `init` or a package-level declaration, is a miss.
- An environment stamp that differs from the current one (D6) is a miss.
- D5's build-failure record beside test entries is no miss.
- A test whose entry records an outcome or a spawn verdict rather than a passing measurement (D4) is no recorded test.

Rationale: own non-test files are instrumented like a linked package's, so D4's function-level argument holds for them;
test files are not instrumented, so only a `Test` function's own text can be matched to one test;
a non-Go file can change any test's path, so it misses as it does for a linked package.
Under the gate, a package the diff touches still runs whole (`gate-scope-coverage`'s D3),
so the own-file rules narrow only a change that reached main after the map's `Commit`.

### D3 — Another package's coverage is recorded by function

A map records, per test, the covered blocks of its own package as today,
and for each linked package the set of functions the test covered,
each named by the linked package's import path, the file and the enclosing-function name the block keys use.
No block key of another package is stored.
A map also records the hash of each of its own package's files other than its Go files:
every tracked file under its directory tree down to the next package directory but the package's own Go files,
`testdata` included, Go files under it among them.

Rationale: the selector matches another package by function and the redundancy step judges own-package blocks only,
so another package's blocks have no consumer, and storing them grows every map and every re-stamp.

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

### D6 — An environment stamp replaces what the age limit stood in for

`testmap-cross-package`'s format marker also holds an environment stamp:
the Go toolchain version, `GOOS` and `GOARCH`, the hashes of `go.mod` and `go.sum`,
a digest of the mapped tiers' command and environment as configured, before the `{parallel}` substitution,
and a digest of the hub `env.yaml` variables the call passes its children.
A stamp that differs from the current one is a miss for the gate (D2) and a whole rebuild for upkeep (D1).

Rationale: a toolchain bump can move block boundaries and a module upgrade changes behaviour with no in-module file changed;
today only the age limit catches either.
A tier's parallel count changes no test's coverage, so it stays out of the digest.

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

### D8 — Upkeep starts on remaining work, not on a landing count

- The prime dobby starts an upkeep pass when no pass is live, no anchor move of the prime stands, and either
  no exit record exists, or the prime's HEAD differs from the planned HEAD in the last exit record, or that record names queued work.
  Queued work is the work left when the pass ended, a map still skipped for a held lock included (D1).
  While the hub's slot count is below two, the dobby starts no pass on queued work alone,
  since such a pass takes no slot (D9).
- The dobby starts a pass as `lym gate testmap --upkeep`, a hidden flag that makes the call a pass;
  it refuses package arguments.
  Under `--upkeep` the verb registers its own process as `runproc.RoleTestMap` under the hub owner,
  in place of the dobby's registration of the process it starts, and refuses to start while another such process lives.
  A `--upkeep` call whose own registration fails reports the failure and exits before it plans;
  the dobby logs that exit once per episode, as it logs a failed tick, and starts a pass again on a later tick.
  The dobby starts no pass while a pass it started is still its live child, registered or not.
  A hand caller may pass the flag too: the call then runs, and writes the exit record, as a dobby-started pass does,
  and the dobby starts no pass while it lives.
- A pass plans once at `H`, then works its queue.
  Before each package job, and on each wake of D9's wait, it reads HEAD; when HEAD moved, it ends, and the next start plans again.
  It ends when its queue is empty and its audit round is done.
- Queued work leaves out a package skipped for a dirty file, a package whose recorded failure names the current HEAD,
  and every spawn-marked test, so neither a dirty prime nor a broken commit restarts passes in a loop.
  A recorded failure is D5's: its failure field, or a failure-only map's `BuildFailure` with its `Commit`.
- Only a call carrying `--upkeep` reads and writes the exit record; any other call leaves it as it was.
- `test_map_batch` and the dobby state's landing count are retired.
  `test_map_every` is retired, and a new dobby key `test_map_upkeep`, true by default, switches passes off when false.
  beam's `configreg.Module.Migrate` rewrites `test_map_every: 0` to `test_map_upkeep: false` and drops any other value.
  It is a migration, not a `configreg.Reinterpretation`:
  `hubreconcile`'s walk runs `configsync.ReconcileHubWideAt`, whose `pruneSparse` drops the units a retired key covers,
  before `applyReinterpretations`, so a reinterpretation never sees the key; `Migrate` runs before that prune.
  beam is a per-worktree module, so the same migration also runs on the prime's `_lym/config/beam.yaml`
  and on beam's local override file when `configsync.ReconcileAll` reconciles those layers.
  So that the switch holds before any walk has run, `configengine.Spec` gains a per-layer migrate hook,
  which `configengine.Load` applies to each layer in memory before it merges the layers and calls `DropRetired`.
  `dobbyengine` owns the rewrite: its `ConfigSpec` sets the hook to it, and beam's `Migrate` calls it.
  The in-memory pass logs nothing; the walk logs the rewrite when it writes it.
  So a prime dobby re-exec'd onto the new binary reads `test_map_every: 0` as `test_map_upkeep: false`.
  `test_map_upkeep` is classified `configengine.CategoryNone`, as `test_map_every` is today, so it loads from the local layer.

Rationale: a backlog drains on its own, which answers `testmap-backlog-drain` by construction;
`test_map_every: 0` is today's off switch, and the migration keeps a hub that set it off, in any layer that set it,
before the walk as after it.
Bound: a pass works one HEAD's plan, so a landing ends it at its next job or wake;
a pass whose dobby died ends the same way;
a call naming no package and lacking `--upkeep` is not registered, so it can run beside a pass;
it wastes work, and the build lock keeps the two off the same package.

### D9 — Upkeep takes a slot only while another stays free, and yields

- A pass takes one gate slot without waiting, and only when at least two slots are free and no drain waits,
  so one stays free after the take.
  `internal/gateslot` exports that take; today it exists unexported in `internal/gateslot/pool.go`.
- While it holds the slot, at each round of runs and at each poll of the pool,
  it checks whether every other slot is held or a multi-slot acquire drains.
  If so, it kills its runs in flight, writes the entries that finished, releases the slot
  and the build lock of the package whose job it interrupted,
  and waits to take a slot again under the same rule.
  With the slot it tries that lock again without waiting;
  when the lock is held, it skips the package as D1 skips a map whose lock is held, and goes on to its next job.
  Holding both, it reads the package's map again and plans it afresh at `H`
  before it runs another test, so a map a hand call wrote meanwhile is not overwritten.
- Its wait wakes at most once a second and backs off while it cannot take a slot (PATTERN-daemon-wakeups).
- On a hub with fewer than two slots a pass takes none: it plans, re-stamps and ends.
- A hand call of `lym gate testmap` naming packages keeps today's blocking take, does not yield,
  waits for each named package's build lock where a pass holds it, and rebuilds the named packages whole.
  A call running under a slot inherited through `gateslot.InheritEnv` neither takes a slot nor yields.
- A call naming no package and lacking `--upkeep` (D8) runs as a pass but leaves the exit record as it was.

Rationale: an arriving gate then waits at most one poll behind upkeep, where today it can wait out a whole refresh.
This hub's `gate.yaml` holds `slots: 3`, so a pass runs beside one gate; the template's `slots: 2` admits it only with no gate running.
Bound: on a hub with fewer than two slots only hand calls naming packages build maps.
A hand call's wait for a build lock ends within one pass job, since a yielding pass gives the lock back.

### D10 — Runs inside a job: side by side where declared safe, written per round

- A job builds the package's coverage binary once at `H` and runs each queued test alone in its own process with its own profile path.
- Tests that call `t.Parallel()` as a top-level statement of their body,
  before any other statement but `tiergate.Require`, run side by side, up to the slot's tier-0 parallel;
  the others run one at a time.
  PATTERN-test-speed has every test open with `tiergate.Require`, so `t.Parallel()` is never a test's first statement.
- The job writes the map after each round of finished runs.
  After each round it reads HEAD and takes `testmap-cross-package`'s dirty read over every linked package;
  when HEAD moved or a linked file is dirty, it discards that round, writes nothing more for the package, and ends the job.
- A terminating signal kills the runs in flight and writes the rounds that finished before it ends.
- Jobs are taken in this order: by the summed per-package elapsed seconds of the retained gate run records (`gateslot.RunRecord`),
  largest first; then every other queued package by import path.

Rationale: a test that declares `t.Parallel()` tolerates its package's other parallel tests in one process,
so separate processes are no riskier; per-round writes make a yield, a signal or a landing cost only the runs in flight.
Every package still gets a map, since the redundancy step reads maps of every package with new tests.

### D11 — An anchor move of the prime ends upkeep instead of waiting for it

`lym fabric anchor`'s in-flight check, `fabriccli`'s `dobbyPassInFlight`,
no longer counts a registered `runproc.RoleTestMap` process as a pass in flight,
so an anchor move of the prime does not wait for a running pass.
The verb's existing end of the prime's hub-owner processes,
`EndDobbies` through `runproc.EndSparing`, then ends the pass.

Rationale: D8 removes the batch cap, so a pass can run for a long time; D10 makes ending it lose only the runs in flight.

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

### D13 — What this changes in the other entries

The three build tasks run the `darn` recipe, whose Darn-Write writes code straight from the stored record.
The parent carries each change below onto the named entry's stored record before that entry's Darn-Write writer starts,
through `lym loom decision add` on the entry's run, or by the record edit task `decision-record-replace` builds once it lands.
`testmap-backlog-drain` and `redundancy-sweep-existing` are notes with no stored record:
the parent carries each one's change below into the note's body.

- `testmap-cross-package`: D3's function sets replace block keys carrying an import path;
  the map records its own package's non-Go file hashes (D3); the format marker holds D6's stamp.
  Its criterion that a linked-only change leaves a map fresh stays, describing the interim rule (D1).
- `coverage-test-select`: the rules of D2, over D3's function sets.
  Its D5 Bound's list of what rebuilds a map becomes D1's plan verdicts at each landing and a hand call naming the package.
- `gate-scope-coverage`: in its D3 union, "every test its map does not record" includes a test whose entry records only an outcome (D4).
- `testmap-backlog-drain`: answered by D8; the note closes when D8's build lands.
- `redundancy-sweep-existing`: before judging a package the sweep runs a hand call naming it (D9), which rebuilds it whole;
  spawn-marked tests are neither candidates nor covers (D4).
- A new note, filed by this run, holds the proposal to narrow the selector's non-body miss by following the names a changed declaration is referenced by,
  with D12's miss counts as the measure of whether it pays.

The parts of this design that call the selector, D1's plan and D7's self-check, order after `coverage-test-select`.

Rationale: the three build tasks were filed on the premise that this design exists, and their records predate it.

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

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - d1-d10-whole-rebuild-write-keeps-stale-entries (class design, severity MEDIUM, open in rounds 2)
  - d8-off-switch-before-walk (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: after a landing that changes the body of a linked function no test of B's map covered, the plan re-stamps B's map to HEAD
  with every entry kept and the marker's `Commit` equal to the map's, and builds and runs nothing; the selector then reads it as no miss.
- D1: after a landing that changes the body of a linked function test T covered, the plan writes B's map at HEAD without T's entry
  and with every other entry, and the job then measures T alone.
- D1: after a landing with a non-body change to a linked file, the plan leaves B's map as it was and queues a whole rebuild of B;
  after the rebuild's first round the map at HEAD holds no entry measured before HEAD.
- D2: the selector called with a commit as target gives the same verdict as with an equal working tree.
- D2: a body change to one of B's own non-test functions selects the tests that covered it and is no miss;
  an edit to a `Test` function's text selects that test alone; an edit to a helper in a test file is a miss;
  an edit to a file under B's `testdata` is a miss.
- D3: a map records, for a test of B that calls a function F of A, F under A's import path and file, and no block of A.
- D4: a test that fails alone gets an outcome entry; a Current verdict keeps it, and a change to one of B's own files drops it.
- D4: a test killed by a yield gets no outcome entry and is queued again,
  and the next pass after one that ended before measuring it still queues it.
- D4: a test the spawn scan marks out-of-process is recorded with its verdict, is never run and leaves the queue empty.
- D4: the redundancy step counts no outcome entry and no spawn-marked test as a cover.
- D5: a coverage build failure at HEAD keeps the package's earlier entries, `Commit` and marker, records the failure with HEAD,
  and the next pass at that HEAD queues nothing for the package; the redundancy step still judges by the kept entries.
- D6: a map whose stamp differs in the toolchain version or the `go.mod` hash is a miss for the selector and is rebuilt whole by upkeep;
  a changed tier `parallel` value changes no stamp; a changed `env.yaml` variable changes it.
- D7: a hub base file holding `test_map_max_age_days` loads after the reconcile walk; no map is rebuilt for its age.
- D7: a kept entry whose block key is absent from a fresh build's block table is re-measured and counted as a self-check mismatch.
- D7: a pass whose plan finds every map current still runs one audit round when it can take a slot,
  and counts a differing result as an audit mismatch.
- D8: on a hub with two or more slots, with no landing since the last pass and an exit record naming queued work,
  the dobby starts a pass on its next tick;
  with no exit record, it starts one.
- D8: a landing ends a pass at its next package job or slot-wait wake; a dirty-skipped or failed-at-HEAD package starts no restart.
- D8: a hand call without `--upkeep` leaves the exit record as it was.
- D8: on a one-slot hub a pass with queued work starts no second pass at the same HEAD.
- D8: a hub base file holding `test_map_every: 0` reads as `test_map_upkeep: false` after the reconcile walk's migration,
  and no pass starts; so does a prime `_lym/config/beam.yaml` or local file holding it; one holding `test_map_batch` loads.
- D8: a prime dobby re-exec'd onto the new binary over a `beam.yaml` holding `test_map_every: 0`, with no walk run, starts no pass.
- D8: a `--upkeep` call whose registration fails exits before it plans, and the dobby logs that exit once per episode.
- D8: while a pass the dobby started lives and has not yet registered, the dobby starts no second pass.
- D9: with one slot free, a pass does not take it; with two free, it takes one.
- D9: when a gate takes the last other free slot, the pass kills its runs, keeps the finished entries and releases within one poll.
- D9: a hand call naming a package a pass is measuring waits for the package's lock, does not yield and rebuilds the package whole.
- D9: a hand call naming a package whose job a pass yielded gets the lock without waiting for the pass to retake a slot.
- D10: tests calling `t.Parallel()` as a top-level statement before any other but `tiergate.Require` run side by side,
  each with its own profile; the map holds the same entries as a serial run of them.
- D10: a HEAD move during a round discards that round and keeps the rounds before it.
- D10: the queue orders packages by summed elapsed seconds from the gate run records.
- D11: an anchor move that finds a pass running ends it and does not wait for its queue.
- D12: the exit record names the planned HEAD, the queued count, the plan's outcome counts with miss reasons and the mismatch counts,
  and the dobby reads it.
- Review: D13 names, for each of the five entries, the change it carries, and the note it files.
