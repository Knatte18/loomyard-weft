## Goal

The prime dobby starts an upkeep pass on remaining work, and an anchor move ends it

Takes: testmap-upkeep-runs' lym gate testmap --upkeep, which registers itself as RoleTestMap and exits on a failed registration, and the exit record's planned HEAD and queued count; today's testMapStep, LandingsSinceTestMap, test_map_every and test_map_batch in dobbyengine, startTestMap in beamcli, dobbyPassInFlight in fabriccli
Delivers: the start rule on no exit record, a moved HEAD or queued work, held off on a hub below two slots for queued work alone, while a pass or its unregistered child lives, and while an anchor move stands; the failed-registration exit logged once per episode; test_map_every, test_map_batch and the landing count retired, the landing count still readable; test_map_upkeep (CategoryNone, true by default); dobbyengine's rewrite of test_map_every: 0 to test_map_upkeep: false, run by beam's configreg Migrate and by a new configengine.Spec per-layer migrate hook that Load applies before merging and DropRetired; dobbyPassInFlight no longer counting a RoleTestMap process
Exports: the dobby key test_map_upkeep and configengine.Spec's per-layer migrate hook

## Scope

The part changes these packages:

- internal/dobbyengine (about 300 changed lines)
- internal/configengine (about 120 changed lines)
- internal/configreg (about 40 changed lines)
- internal/beamcli (about 120 changed lines)
- internal/fabriccli (about 40 changed lines)

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

### D11 — An anchor move of the prime ends upkeep instead of waiting for it

`lym fabric anchor`'s in-flight check, `fabriccli`'s `dobbyPassInFlight`,
no longer counts a registered `runproc.RoleTestMap` process as a pass in flight,
so an anchor move of the prime does not wait for a running pass.
The verb's existing end of the prime's hub-owner processes,
`EndDobbies` through `runproc.EndSparing`, then ends the pass.

Rationale: D8 removes the batch cap, so a pass can run for a long time; D10 makes ending it lose only the runs in flight.

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

- D8: on a hub with two or more slots, with no landing since the last pass and an exit record naming queued work,
  the dobby starts a pass on its next tick;
  with no exit record, it starts one.
- D8: on a one-slot hub a pass with queued work starts no second pass at the same HEAD.
- D8: a hub base file holding `test_map_every: 0` reads as `test_map_upkeep: false` after the reconcile walk's migration,
  and no pass starts; so does a prime `_lym/config/beam.yaml` or local file holding it; one holding `test_map_batch` loads.
- D8: a prime dobby re-exec'd onto the new binary over a `beam.yaml` holding `test_map_every: 0`, with no walk run, starts no pass.
- D8: a `--upkeep` call whose registration fails exits before it plans, and the dobby logs that exit once per episode.
- D8: while a pass the dobby started lives and has not yet registered, the dobby starts no second pass.
- D11: an anchor move that finds a pass running ends it and does not wait for its queue.
