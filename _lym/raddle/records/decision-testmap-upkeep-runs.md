## Goal

An upkeep pass runs under a slot it yields to a gate, side by side where tests declare it, and writes per round

Takes: testmap-upkeep-plan's plan, queue, job and exit record; gateslot's unexported tryAcquireSlot, the pool's drain and gateslot.RunRecord
Delivers: the verb side of D8: the hidden --upkeep flag refusing package arguments, its own RoleTestMap registration under the hub owner, refusal beside a live pass, exit before the plan on a failed registration, the exit record read and written only under --upkeep, HEAD read before each job and wake, queued work leaving out dirty-skipped, failed-at-HEAD and spawn-marked work; the take only while two slots are free and no drain waits, the yield that kills runs, writes finished entries and releases the slot and lock, the wait at most once a second with backoff, the lock retry and fresh plan on retake, no slot on a hub below two, and hand calls' blocking take, lock wait and whole rebuild (D9); side-by-side runs for top-level t.Parallel() tests up to the tier-0 parallel, per-round writes, the per-round HEAD and dirty read, signal handling and the job order from gate run records (D10)
Exports: lym gate testmap --upkeep, a registered pass that exits on a failed registration, which testmap-upkeep-dobby starts; gateslot's exported non-blocking take

## Scope

The part changes these packages:

- internal/gatecli (about 700 changed lines)
- internal/gateslot (about 50 changed lines)

## Decisions

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

## Acceptance criteria

- D4: a test killed by a yield gets no outcome entry and is queued again,
  and the next pass after one that ended before measuring it still queues it.
- D8: a landing ends a pass at its next package job or slot-wait wake; a dirty-skipped or failed-at-HEAD package starts no restart.
- D8: a hand call without `--upkeep` leaves the exit record as it was.
- D9: with one slot free, a pass does not take it; with two free, it takes one.
- D9: when a gate takes the last other free slot, the pass kills its runs, keeps the finished entries and releases within one poll.
- D9: a hand call naming a package a pass is measuring waits for the package's lock, does not yield and rebuilds the package whole.
- D9: a hand call naming a package whose job a pass yielded gets the lock without waiting for the pass to retake a slot.
- D10: tests calling `t.Parallel()` as a top-level statement before any other but `tiergate.Require` run side by side,
  each with its own profile; the map holds the same entries as a serial run of them.
- D10: a HEAD move during a round discards that round and keeps the rounds before it.
- D10: the queue orders packages by summed elapsed seconds from the gate run records.
