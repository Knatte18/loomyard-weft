# Decision record — gate-builds-testmap

## Goal

Building the coverage maps becomes a mode of `lym gate test`, set by a `--map` flag over the tier 0 and tier 1 tests,
and the `lym gate testmap` verb goes with its planning, audit round, build locks, waits and package ordering.
In map mode the gate builds each package's test binary with `-cover`, runs each test as its own process from it,
in parallel within the gate's own parallelism and memory bounds, and `internal/testcover` merges the per-test profiles into the package's map.
The maps keep their format, their location and their consumers; only the way they are built changes.

## Scope

In:

- A `--map` flag on `lym gate test` and the map run behind it.
- Removing the `lym gate testmap` verb and every piece only it reaches:
  incremental planning, the audit round, the self-check of kept entries, the per-package build lock and its wait, `--max`, and the run-record ordering.
- Code in `internal/testcover` that only the retired planning reaches goes, per `PATTERN-test-only-code`.
- Every live text naming the verb: `README.md`, `docs/overview.md`, `internal/gatecli/doc.go`, `internal/gateslot/doc.go`,
  `internal/dobbyengine/doc.go`, `internal/gateslot/template.yaml`'s comments,
  and the backgrounds `pattern/PATTERN-test-speed.md`, `pattern/PATTERN-daemon-wakeups.md` and `pattern/PATTERN-gate-slot.md`.
- The way-forward text of `lym gate redundant`'s refusals that name a map rebuild.

Out:

- What the maps are used for: test selection, redundancy and review use are decided apart from this task.
  `lym gate redundant` and the gate's `redundancy` report step keep reading maps as they do.
- The map file format and the maps directory.
- tier2 and tier3 tests: no map holds them, as today.
- A map built during an ordinary gate run.
- Research reports under `docs/research/`, which stay as historical records.

## Decisions

### D1 — `--map` is a mode of `lym gate test` over tiers 0 and 1

`lym gate test --map [<targets>...]` builds the maps of the call's packages.
The scope is chosen as for any call: explicit targets, `--all`, `--card`, `--base`, or the diff from the merge base.
The package set is every package of that scope that has a map or holds a tier 0 or tier 1 test.
A package with no mapped test gets a map of its file hashes with no tests, as today.
The call measures the tier 0 and tier 1 tests of each package in one pass, so each map holds both tiers' entries.
`--map` refuses `--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `--dir`/`-C` and words after `--`,
each refusal naming the way forward: `lym gate test --map` with targets or a scope flag.
`--main-packages` applies with `--card` as for any call.
It refuses a hub whose tier 0 or tier 1 is not a `go-json` tier, naming the tier, since a coverage map exists only for Go.
It also refuses a hub whose tier 0 or tier 1 sets `memory_mb` below `redundancy_test_memory_mb`, per D4.
A hub without a tier 1 maps tier 0 alone.
Agent calls without `--wait` go through the detached runner like any other gate call;
`--map` is one of the words the detached call is keyed on and replayed from, so a map call never joins a plain call's run, nor a plain call a map run.

Rationale: the operator's idea makes the map a mode of the gate's own tier 0 and tier 1 run.
One map holds both tiers, so measuring them in one pass keeps a map whole instead of half-rewritten by a one-tier call.
Bound: map mode skips the tiers' commands, their `filter`s, their `memory_mb` scopes,
their `timeout_min` and `test_timeout_min`, and the shared build step,
and it runs no lint, report, guard step, carry or card fixer.
What bounds that is that the run is on command only and writes nothing but maps and its log.

### D2 — A map run rebuilds each package's map whole

Map mode has no incremental planning: each package in the call is measured whole and its map is replaced whole,
except that a failed build keeps the old entries per D7.
There is no audit round, no self-check of kept entries, no per-package build lock, no lock wait, no `--max` and no ordering by run records.
Packages are measured one after another in import-path order;
the next package's coverage build may start while the current package's tests run, one package ahead.
A call that ends early cancels that pending build and waits for it.

Rationale: the smart-build study (`docs/research/speed/testmap-smart-build-2026-11-15/report.md`, Go 1.26.8, 12 CPUs)
put a whole build of every map at about 16 CPU-minutes once the measure loop is lean, so whole rebuilds are affordable on command.

### D3 — Each test runs as its own process from one coverage binary

Per package the gate builds one test binary with `go test -c` in `set` cover mode, with the in-module packages it links as the cover set, as the verb does today.
It runs each mapped test alone through `-test.run` with its own `-test.coverprofile`, under its tier's `env` (`LYM_TEST_TIER` included).
Each coverage build is bounded by `redundancy_build_timeout_min`,
and each test process by `redundancy_test_timeout_sec`, as the verb's are today;
a build past its bound is a failed build and a test past its bound a timed-out test.
Map mode builds no `lym`: `LYM_PREBUILT_LYM`'s one reader is `internal/testkit/lymbin`,
a tier0 test spawns nothing, and `testcover.ScanSpawns` marks every other test that reaches it as spawn-marked,
so no test the run measures reads that variable.
`internal/testcover` reduces each profile and merges the runs into the package's map, written once when the package ends.
Spawn-marked tests, which may run module code in another process, are not run and get an entry with their spawn verdict, as today.
A skipped, failed, timed-out or killed test gets an entry with that outcome and no blocks.

Rationale: separate processes keep separate counters, which is the one way to get per-test coverage from Go's block counters.

### D4 — Test processes run side by side within the gate's parallelism and memory bounds

Within a package, a tier's tests run side by side up to that tier's `parallel` value, with the per-machine `tier_parallel` override applied.
A package's tier 0 tests run first, then its tier 1 tests, each tier up to its own clamped count, so at most one tier's count runs at once.
Each test process runs under the `redundancy_test_memory_mb` kernel cap.
The count is clamped to memory as `lym gate redundant` clamps it today: the caps of the processes side by side stay within the headroom lym's cgroup leaves.
When the tier sets `memory_mb`, the count is also at most `memory_mb` divided by the per-test cap, whether or not lym's cgroup is limited,
so the tier's cap bounds the run's test processes together on a host whose memory cap covers a process tree;
on one where it does not, one test runs at a time and each process is capped alone, as the tier's own `memory_mb` is there.
A map call refuses a hub whose tier 0 or tier 1 sets `memory_mb` below `redundancy_test_memory_mb`, since that quotient is 0;
the refusal names the tier and both values,
and its way forward is to raise the tier's `memory_mb` or lower `redundancy_test_memory_mb` with `lym config gate`.
The memory guard samples the map run's own process tree, as `lym gate testmap` does today, so a `.test` process past `memory_quit_mb` is killed and named.

Rationale: the operator asks that the processes be bounded like the gate's own parallelism and that the guard and the per-run cap cover them;
on a host whose memory cap covers a process tree,
clamping the per-process caps to the tier's `memory_mb` gives the same total as a scope around them.
Bound: the coverage builds run outside the per-test cap and outside the tier's `memory_mb`;
they are `go` builds bounded by tier 0's `parallel` as their `-p`, as the verb's builds are today.

### D5 — A test that fails beside others reruns alone, and the report names it

A test that fails, times out or is killed by the guard or its cap while others run beside it runs once more alone, and that outcome stands.
The call's report lists every test that passed only alone, with its package.

Rationale: tests of one package can share state outside the process, and the guard's pressure trigger can kill a test for its siblings' memory;
the known cross-process collision is `tmuxkit.Main`'s check for a `.git` in the shared scratch root.
The rerun keeps a collision from recording a false failure, and the list makes each collision visible so its test can be fixed.
Bound: a test that passed only alone enters the map as a plain pass, and the list in the call's output and log is the only trace,
so a flaky test can stand as a cover until its collision is fixed.

### D6 — A map is written only when its files are clean and HEAD is still

Once a package's coverage build has listed the linked packages and ended, compiled or failed, and again before its map write,
the run checks that HEAD is the commit it read at the start,
and that no file the package or any in-module package it links owns is modified or untracked,
a package owning the files under its directory down to the next package directory or nested module.
A package with no mapped test has no build, so its one check runs before its map write.
A build whose listing of the linked packages fails is checked before its failure is written,
for HEAD and for every modified or untracked file of the module, since no linked set is known;
a failed check leaves the map unwritten and lists the package as unmeasured, as below.
When either check fails, the package's map is not written and the package is listed as unmeasured with the reason, naming the dirty files.
A moved HEAD ends the call; a dirty package does not, and the call goes on to the next package.
Either case makes the call exit non-zero.

Rationale: a map records the commit and the file hashes it was built from, and its consumers trust both;
a check per package, after its build and before its write, catches an edit still in place at either check,
where a check only at the start would miss every edit made after it.
Bound: an edit made before a package's first check, or between its two checks, and reverted before the next check goes unseen,
so the map holds coverage of the edited code under the committed hashes.
A gitignored file in a package directory is not seen by either check,
so a gitignored Go file the build compiles enters the map under HEAD's hashes.

### D7 — A map run's slots, records and exit

A map run holds, for the whole call, as many gate slots as the larger of tier 0's and tier 1's slot asks,
where a tier asking for every slot ranks above any count.
It writes a log, and no run record, green record, verify record or failure record:
`lym gate report` totals no map run, and no later gate call skips a tier because of one.
A map run subtracts no green tier either, so a green record never stands in for a measurement.
It exits non-zero when any test's outcome after D5's alone-rerun is a failure, a timeout or a kill, when a package's build failed, or per D6;
it still writes the map of every package D6 lets through, failed entries included.
A failed build records the failure on the package's map and keeps its old entries, as today.

Rationale: a map run is a measurement, not a verdict on the tree:
tests run under coverage and one per process, so a pass there says nothing a tier run would record.
A non-zero exit still tells the caller its map holds failed entries.
Bound: on a hub whose tier asks `slots: all`, a map run holds every slot for its whole length, minutes for `--all`, and every other gate on the hub waits;
the run is on command only, so the operator chooses when to pay that.

### D8 — The maps keep their place and format

Maps are written where `lym gate testmap` writes them today:
the hub's shared maps from the prime worktree, and the worktree's anchor `.lym` from any other worktree.
The map format stays as it is, written through `internal/state` under its lock.
Two map runs over the same package from the same worktree each write their map at once, and the later write wins.
The old per-package `.build.lock` files left beside existing maps are inert; no reader opens them.

Rationale: `lym gate redundant` and the report step read these places today, and nothing in this task changes what they read.
When both builds succeed, the last-write-wins race leaves a whole map from one measurement, since each write replaces the map at once;
a failed build's write keeps the entries it read, which may come from the other run.

### D9 — `lym gate testmap` goes with what only it reaches

The verb, its files under `internal/gatecli`, the planning, audit and lock code they reach, and the measure loop's `lym` build go.
Every refusal of `lym gate redundant` that names a map rebuild names `lym gate test --map <package>` from the worktree as its way forward:
a map missing, unreadable, recording only a failed build, or built from other Go files than the worktree holds.
The last one's way forward also says to commit or revert the worktree's edits first, since D6 measures no package with a modified file.
Every live doc naming the verb names the flag instead.

Rationale: one way to build a map, through the gate, is the simplification the task asks for.

## Constraints

- `PATTERN-gate-slot`: a map run is a gate test run, so it holds slots and keeps `-trimpath` in its Go flags;
  it adds no non-tier holder and removes the testmap holder from the list in `internal/gateslot`'s doc.
  The background's slot rule names the map run of `lym gate test --map` as a third place a gate build or test runs,
  holding the larger of tier 0's and tier 1's slot asks, where a tier asking for every slot ranks above any count.
- `PATTERN-test-speed`: tests still run only through `lym gate test`, and the entry's overview line does not change.
  Its background's redundancy-run paragraph names the flag, and its coverage-map section is rewritten for D2 to D7:
  the planning, self-check and audit sentences go.
- `PATTERN-daemon-wakeups`: the map build lock wait goes, so its background line and the `cmd/lym/daemonwakeups_test.go` row naming `internal/gatecli/testmapwait.go` go too.
  The memory guard's one-second sampling stays.
- `PATTERN-child-processes`: every test process and build is logged at spawn and waited on by the map run, a cancelled one included.
- `PATTERN-refusal-way-forward`: every refusal D1, D4 and D9 add names its way forward, and a test reaches it.
- `PATTERN-code-conventions`: map mode is tied to a tier's `go-json` counter in config, not to a hard-coded language check.
- `PATTERN-tolerant-records`: the map format is unchanged, so old maps still load.
- `PATTERN-test-only-code`: functions only the retired planning, audit, wait or ordering reached go, in `internal/gatecli` and `internal/testcover` alike.
- Fact, from the smart-build study at Go 1.26.8: a `set`-mode coverage build reuses the instrumented packages a previous coverage build cached,
  while the instrumented packages share no cache entry with the normal gate build.
  A map run pays for instrumenting its cover set once, then reuses it across worktrees through `-trimpath`.

## Auto-mode assumptions

- The flag is named `--map`.
- The rerun-alone list is the only collision handling; no test is marked serial-only.

## Open risks

- A test that writes the real home directory, a repository file or another fixed path can collide with its siblings in ways the alone-rerun hides as a pass.
  The exploration found real-home reads only in smoke tests outside the mapped tiers; the D5 list is how a new case shows up.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - d6-failed-listing-written-unchecked (class design, severity MEDIUM, open in rounds 2)
  - map-run-time-bounds-unstated (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: `lym gate test --map <pkg>` writes the package's map holding an entry for each of its tier 0 and tier 1 tests, each with its own blocks.
- D1: `--map` with `--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `--dir` or words after `--` is refused with a message naming `lym gate test --map` with targets as the way forward.
- D1: `--map` on a hub whose tier 0 or tier 1 is not a `go-json` tier is refused, naming the tier.
- D1: a detached `--map` call and a plain call over the same targets get different detached runs, and the detached map call writes the map.
- D2: a map run over a package whose map already exists replaces every entry, so an entry of a deleted test is gone.
- D2: a map run over a package whose last mapped test was deleted rewrites its map with no test entries.
- D3: two tests of one package measured side by side each get only the blocks they cover alone.
- D3: a spawn-marked test is not run and gets an entry with its spawn verdict; a skipped test gets a skipped entry with no blocks.
- D4: the number of test processes side by side never passes the running tier's `parallel`, nor its `memory_mb` divided by the per-test cap when it sets `memory_mb`.
- D4: a map call on a hub whose tier 0 or tier 1 sets `memory_mb` below `redundancy_test_memory_mb` is refused,
  naming the tier, both values and `lym config gate` as the way forward.
- D4: a test process the memory guard kills is named in the run's output, and the run goes on with the package's other tests.
- D5: a test that fails or is killed beside others and passes alone gets a passing entry and is listed in the call's report.
- D6: a package with a modified file the package or a linked package owns gets no map written, is listed as unmeasured naming the file, and the call exits non-zero.
- D6: a package whose listing of its linked packages fails through an uncommitted edit, in its own files or a linked package's,
  gets no map written, is listed as unmeasured naming the file, and the call exits non-zero.
- D6: when HEAD moves during a map run, the current package's map is not written and the call exits non-zero, naming the unmeasured packages.
- D7: a map run holds gate slots while it measures, writes no run record, and a gate call after it over the same tree, with no green record from before the map run,
  runs its tiers rather than skipping them as green.
- D7: a map run with a test that fails alone exits non-zero and still writes the package's map with the failed entry.
- D7: a package whose coverage build fails keeps its old entries and records the build failure.
- D8: a map run from the prime writes the hub's shared maps, and one from another worktree writes that worktree's anchor `.lym`.
- D9: `lym gate testmap` is an unknown command.
- D9: `lym gate redundant <pkg>` with no map, or with a map recording only a failed build, is refused, naming `lym gate test --map <pkg>`.
- Review: D9, `README.md`, `docs/overview.md`, the package docs and the three PATTERN backgrounds name `lym gate test --map` and no longer name `lym gate testmap`,
  and `pattern/PATTERN-gate-slot.md` names the map run as holding the larger of tier 0's and tier 1's slot asks,
  where a tier asking for every slot ranks above any count.

## Notes for the plan writer

- Most of the measure loop exists in `internal/gatecli/testmappackage.go`: the coverage build, the side-by-side runs, the alone-rerun, the profile reduction and the map write.
  The task is to move it under `lym gate test` and drop the planning around it, not to write it afresh.
