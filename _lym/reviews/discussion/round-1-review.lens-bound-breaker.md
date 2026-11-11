```yaml
lens: bound-breaker
outcome: reported
```

**bound-breaker: claims attacked**

1. D8: "A `configreg.Reinterpretation` rewrites a hub's `test_map_every: 0` to `test_map_upkeep: false`." **Broken**: the walk prunes retired keys before it applies reinterpretations.
2. D8 Rationale: "the reinterpretation keeps a hub that set it off." **Broken**: beam is a per-worktree module, and the reinterpretation reaches the hub base file only.
3. D9: "On a hub with fewer than two slots a pass takes none: it plans, re-stamps and ends." D8 says every pass ends "when its queue is empty and its audit round is done". **Broken**: a dobby restart loop.
4. D9: "A hand call … waits for each named package's build lock where a pass holds it." **Broken**: a deadlock on a two-slot hub.
5. D1 Current verdict together with the D4 criterion "a test killed by a yield … is queued again" and D8's "a backlog drains on its own". **Broken**: a gap left by a pass that a landing ended is never refilled.
6. D10: "Tests whose top-level body opens with `t.Parallel()` run side by side." **Broken**: no test in the repository opens with it.
7. D6 and D7 Rationale: "D6 bounds the environment." **Broken in part**: env.yaml variables and the caller's process environment reach tests but are not in the stamp.
8. D1: "skips a map whose lock is held; the next pass plans it again." **Broken in part**: with no landing and no queued work, no next pass starts.
9. D8: "Only a pass the dobby starts reads and writes the exit record; a hand call leaves it as it was." **Unbounded**: a bare `lym gate testmap` with no package named is neither a hand call by D9's wording nor a pass the dobby started.
10. D1 re-stamp: "every entry is kept" with own-package block keys after a body change elsewhere in the file. **Held**: `BlockKey` keys a block by its content, not its position (`internal/testcover/blockkey.go:22-32`).
11. D11: ending a pass through `runproc.EndRole`. **Held**: it ends the processes below the record too, with SIGTERM and then SIGKILL after the grace (`internal/runproc/end.go:71-77`).
12. D11 premise: today an anchor move waits for a running refresh. **Held** (`internal/dobbyengine/doc.go:500-502`).
13. D9 premise: `tryAcquireAny` is unexported, and `Holders` and `Draining` are exported. **Held** (`internal/gateslot/pool.go:301,338,374`).
14. D9 Rationale: this hub holds `slots: 3` and the template holds `slots: 2`. **Held** (Bolt `gate.yaml:1`, `internal/gateslot/template.yaml:1`).
15. D10: run records give per-package elapsed seconds. **Held** (`PackageLaunch.ElapsedSeconds`, `internal/gateslot/runlog.go:113-117`). Packages that have none fall to import-path order.
16. Constraint: a strict load does not refuse a retired `test_map_max_age_days`. **Held**: a load drops `RetiredKeys` (`internal/configengine/layers.go:81-84`).
17. D5 premise: `BuildFailure` makes the redundancy step refuse the map. **Held** (`internal/gatecli/redundancy.go:305`).
18. D4 Rationale: "costs gate time, never a missed test" for outcome entries. **Held** as a cost-only claim.

**Findings**

- **BLOCKING · design · D8 and Constraint PATTERN-config-layers.** The reinterpretation of `test_map_every: 0` never fires, because the same walk first drops the retired key from the hub base file.
  - `reconcileHubWide` runs `configsync.ReconcileHubWideAt` (`internal/hubreconcile/walk.go:232`) before `applyReinterpretations` (`walk.go:254`).
  - For a present base file of every module, `ReconcileHubWideAt` calls `pruneSparse` (`internal/configsync/configsync.go:1157`). Without `PruneUnknown`, that removes every unit a retired key covers (`configsync.go:719-739`, `DropRetired`).
  - So `test_map_every: 0` is gone before the rewrite reads the file, and upkeep switches on for a hub that had it off.
  - A `Module.Migrate` runs before the prune (`configsync.go:1140`), and its doc says it rewrites retired keys (`internal/configreg/configreg.go:77-78`). The record names the wrong mechanism.

- **MEDIUM · design · D8.** The reinterpretation reaches only the hub base file, `configengine.ConfigFile(boardDir, module.Name)` (`internal/hubreconcile/reinterpret.go:50`).
  - beam is a per-worktree module (`internal/dobbyengine/config.go:54-58`). The dobby's load stacks the prime worktree's `_lym/config/beam.yaml` and the local file `.lym/config/beam.local.yaml` above the hub file (`internal/configengine/layers.go:131-146`).
  - `test_map_every` is `CategoryNone`, so the local layer may hold it.
  - A `test_map_every: 0` in either layer is dropped as a retired key on load, and passes start. The record states no bound for this.

- **BLOCKING · design · D8 and D9.** On a hub with fewer than two slots, every pass ends with work still queued. Its exit record therefore names queued work.
  - D8 starts a pass whenever "that record names queued work". D8's exclusions cover dirty-skipped packages, failed-at-HEAD packages and spawn-marked tests, not a slotless hub.
  - So the dobby starts a pass on every tick, and each pass re-plans every map through `CodeFileAt`, with no end.
  - D9's Bound ("only hand calls build maps") does not cover this, and D7's audit never runs there either.

- **BLOCKING · design · D9 and D13.** A hand call can wait forever on a build lock that a yielded pass holds.
  - On the template's `slots: 2`, the pass yields when a gate takes the other slot. It then waits for two free slots while the record leaves it holding package B's build lock.
  - A hand call naming A and B takes a slot once the gate ends. Today it holds that slot to the end of the call (`internal/gatecli/testmappackage.go:113-139`), so it builds A and then waits for B's lock while holding its slot.
  - The pass can never see two free slots, and the hand call never gets B's lock. Nothing in D8 or D9 ends this without a landing.
  - The record states neither whether a yielding pass gives up its build lock nor any bound on the hand call's lock wait. D13 makes this hand call the redundancy sweep's step.

- **MEDIUM · design · D1, with the D4 and D8 criteria.** D1 queues "every declared mapped test with no entry" only under the Affected-tests verdict. A Current verdict queues nothing.
  - Counterexample: a pass drops test T as affected at H1 and writes the map at H1. A landing elsewhere then ends the pass before T is measured (D8).
  - The next plan at H2 finds B Current and re-stamps it. T stays without an entry until some later change marks a test of B affected or misses.
  - This contradicts the criterion "a test killed by a yield … is queued again" once a pass ends rather than waiting. It also breaks D8's claim that a backlog drains on its own.
  - At about 173 landings a week, such gaps pile up. The map stays sound, but the gate pays for the gaps.

- **MEDIUM · design · D10.** "Tests whose top-level body opens with `t.Parallel()`" matches no test here.
  - PATTERN-test-speed requires every `Test` function to open with `tiergate.Require(t, N)` (`pattern/PATTERN-test-speed.md:22`).
  - Of the top-level tests under `internal/`, 0 have `t.Parallel()` as their first line and 5198 have `tiergate.Require` (grep).
  - So D10 runs every test serially. The D10 criterion and the Rationale's speed-up rest on a rule that selects nothing.

- **LOW · design · D6 and D7 Rationale.** The stamp covers the tiers' configured command and env only.
  - testmap's children also get the hub `env.yaml` variables and the caller's environment, through `gateEnv(r.request.environ, hubEnv)` (`internal/gatecli/testmap.go:265`).
  - A change to those can change any test's path with no miss, which today only the age limit catches.
  - So "D6 bounds the environment" overstates what the stamp sees.

- **LOW · design · D1.** "Skips a map whose lock is held; the next pass plans it again" holds only if a next pass starts.
  - D8 starts one only on a HEAD change or queued work in the exit record, and the record does not say whether a lock-skipped map counts as queued.
  - With no landing, the skipped map waits for the next landing.

- **LOW · design · D8 and D9.** The record names no mechanism by which the verb tells a dobby-started pass from a hand call.
  - A bare `lym gate testmap` from the operator names no package, so D9 does not treat it as a hand call. It is also not dobby-started, so D8 does not say whether it writes the exit record or yields.
  - Whatever marker distinguishes the two, such as a flag, is a new edge whose bound the record does not state.

**Tool calls**
- `internal/testcover/blockkey.go` 1–60: whether a block key is positional, to test D1's re-stamp claim. The manifest did not include it.
- grep of the first line of `func Test` bodies under `internal/`: to test D10's "opens with `t.Parallel()`". Not in context.
- `internal/runproc/end.go` 60–130: `EndRole`'s end sequence, for D11. Context held only the grep line.
