# Each test runs once per tree: disjoint tiers, row entry gates, carried green runs, tmux in final

`gate-runs-once` · Task · enhancement, gate, shed, loom, webster, fabric, batten · running · Webster

## Order (operator, 2026-10-23)

This task runs after `loom-tail-review`, which rebuilds the run tail: Publish, PR-Align, PR-Gate, PR-Rework and Merge-Review go, and Publish's gates move into Merge-In and Finalize.
The rows and gates named below were written against the old tail; map each onto the new tail as it landed.

## One gate key per producer (operator decision, 2026-10-23)

Every producer that runs a test gate gets its own key in `gate.yaml`'s `gates`, named after that producer, listing the tiers it runs.
No generic gate kind stays: `landing` goes, since `Land` is batten's row and `landing.yaml` a config module, and neither runs that gate; `final` goes the same way.
After `loom-tail-review` the generic `landing` call runs at Merge-In, Finalize, Darn-Write, Darn-Burler-Fix and Code-Burler-Fix, and Webster's end gate is `final`; each such site gets its own key.
`card` and `batch` already name where they run and stay; whether `round` stays under that name is weighed by the same rule.
This lets an umbrella's piece runs (note `loom-beam-darns`) run fewer tiers at their writer than a standalone darn, with no change to Merge-In or Finalize.
`lyx config reconcile` migrates a hub's `landing`, `final` and `round` lists onto the new keys.

Each key also sets its scope, not only its tiers (operator, 2026-10-24): today every call site hard-codes what its gate runs over (`--card`, the diff, `--all`).
Scope values such as `card`, `diff` (since the last green run, or since the fork from the parent) and `all`; a site refuses a scope it has no data for, such as `card` outside Webster.

## Darn and loom symmetric (operator, 2026-10-23)

Which tiers run at which row should be symmetric between darn and loom, and stated plainly for each recipe.
Seen on `status-overview-trim` (darn), before `loom-tail-review` landed: Darn-Write's gate ran the `round` call, tiers 0 and 1 scoped by `--base`, and Merge-In then ran `landing`, tiers 0, 1 and 2 over the whole tree, though Merge-In changed nothing, since the parent had not moved.
`loom-tail-review`'s D8 moves the darn writer and fix rows onto the `landing` call through `verifytree.Verify`, which skips a tree it already verified; check what is left of this once it lands.
In loom, Webster's `final` gate runs the same tiers as `landing` over the same tree before Merge-In.
The piece runs of an umbrella (note `loom-beam-darns`) run lighter tests than a standalone darn; that note holds that part.

## Decision (operator, 2026-10-22)

Three notes merged into one loom task, so each test runs once per tree and in one tier.
Scope:

1. Disjoint tiers: every tier is tagged by its own name, `tier0` included, so no test runs in two tiers.
2. Row entry gates in shed, as in the orch recommendation below: a row may name an entry check, and a skipped row is recorded with its reason.
   The switched producer's flag check and `internal/loomrecipe`'s darn edge rewrite move onto it.
3. A green run carries forward: a verify row's entry check skips it when the same tiers were green over the same tree.
4. Done by hub config on 2026-10-23: the `final` gate runs `tier2`, the tmux tier, too.
5. Measure a loom run's gate time per gate kind before and after, including the Cortex XDR hold's share on OSL-1046.

Speeding up `internal/fabricengine`'s tests is out of scope.

## Test runs repeat (from note gate-test-repeats)

### Seen (2026-10-22)

The loomyard hub's tiers were first declared on Bolt on 2026-10-22 (`d5a60908b`): `tier0` untagged, `tier1` integration (with `go build ./...` and `go vet` under the integration, tmux and llm tags first), `tier2` tmux, `tier3` llm by hand only.
The gate lists are `card: [0]`, `batch`, `round`, `final` and `landing: [0, 1]`, and `publish: [2]`.

Three things make a loom run's test runs clumsy:

- **`tier0` runs again inside every higher tier.**
  A Go build tag only adds files, so `go test -tags integration` and `go test -tags tmux` both compile and run every untagged test; `covers: [0]` only stops a call from running `tier0` a separate time.
- **The whole suite runs up to four times at the end of a loom run.**
  Webster's `final` gate, Merge-In's `landing` gate, Publish's `landing` gate plus `publish`, and Finalize's `landing` gate each run `tier0` and `tier1` over the whole tree.
  Since `905bc88df` a re-run over an unchanged tree should take its results from Go's test cache, so only a tree that changed, such as after Merge-In takes a moved parent, runs for real; how close to free that is in practice is not measured, and each run still waits for a slot, starts up and writes a log.
- **The tmux tier runs only at Publish.**
  `PATTERN-test-speed` gives it its own tier because it needs a real tmux server and was slower, but the measurement below has it faster than `tier1`.

#### Measurement (fork, 2026-10-22, Neve, `de6b8465f`)

- `tier1 --all`, cold (first run of the commit on the machine): 114 s wall; about 70 to 75 s of it was build, the three vets and test compiles.
- `tier1 --all`, fully cached: 28.8 s, about 20 s of it the fixed build and vet cost.
- A normal `tier1` run after a change: about 60 s, about 20 s build and vet plus about 40 s tests.
- `tier1` package time 125.9 s at `-p=4`: about 20 s is the tests `tier0` also runs, about 106 s the integration-only tests.
- `tier0 --all`: 22.5 s wall.
- `tier2 --all`: 60 s wall with a warm build cache, about 45 s of it tests.
- Slowest `tier1` packages: `internal/fabricengine` 37.2 s, then `battencli` 8.8 s, `cmd/lyx` 6.9 s, `gatecli` 6.4 s, `webstercli` 5.8 s.
  `fabricengine` alone sets the length of `tier1`'s test phase.
- Slowest `tier2` packages: `loomcli` 44.7 s, `reedengine` 37.9 s, `reedcli` 37.4 s.

### Idea

- **Disjoint tiers.**
  Mark `tier0` test files so a higher tier does not run them again.
  Orch recommendation, not a decision: a negated tag such as `//go:build !integration && !tmux`, which plain `go test` and gopls still see, over a positive `//go:build tier0`, which hides the tests from both; either touches every untagged test file, and a new tier tag must join the negation.
  The measured gain is small, about 20 s of package time per higher-tier run.
- **A green run carries forward.**
  A gate runs its tiers only when the tree changed since the last green run of the same tiers, instead of relying on Go's test cache to make the repeat cheap.
- **Run tmux more often.**
  The tmux tests run in a batch or final gate, or share a tier with integration, instead of only at Publish.
  The tmux tests have been the flaky ones in gates (task `darn-verify-fixes`), which a move must weigh.
- **Speed up `internal/fabricengine`'s tests**, the largest single item in `tier1`.

### Operator view, not a decision

- The operator pictured every tier tagged by its own name, `tier0` included, so no test runs in two tiers (2026-10-22).
- tmux is an important part of this project, so running its tests less often is not good (2026-10-22).
- Suites running several times at the end of a loom run is clumsy (2026-10-22).

## Row entry gate (from note row-entry-gate)

### Seen (orch, 2026-10-17)

Two ways exist today to leave out a row that is not wanted:

- Darn turns its review perch off by a recipe rewrite: with `review_max_bounces` 0, `internal/loomrecipe` rewrites Darn-Write's Done edge straight to Merge-In when the recipe loads (`contracts/recipes/darn-recipe.yaml`), so the perch rows are never entered.
- Batten's Open-Window, Seed-IDE and Open-IDE rows are `shedadapters`' switched producer: the row reads its `batten.yaml` flag and is Done without acting when the flag is off.

### Operator view, not a decision (2026-10-17)

The recipe rewrite is a terrible solution.
A Bouncer should instead have an entry gate: a check that runs first and decides whether the Bouncer runs at all.
For darn's perch that check is only whether `review_max_bounces` is above 0, and the producer that opens VS Code works the same way: it checks its flag and skips everything when the flag is off.

### Orch recommendation, not a decision

- The entry gate belongs in shed, for every row, since every FSM needs it: a row in the recipe may name an entry check, read from the run's config stack when the row is reached; when it is false, the row is Done without acting and the run follows its Done edge.
  A perch's Burler row is reached only through the Bouncer's stuck edge, so a skipped Bouncer skips the whole perch.
- Status and the run's history record the row as skipped with the check's reason, so a skip is visible.
- The switched producer's flag check becomes this gate, and `internal/loomrecipe`'s edge rewrite for darn is removed.
- The regression perch (note `regression-baseline-verify`) and the `Command` rows (task `batten-start-speed`) use the same gate.

### Where it sits (2026-10-17)

Operator view, not a decision: the recipe rewrite is not clean at all; every producer should be able to carry an entry gate, set in the one template all producers share.

Orch recommendation, not a decision, from the code:

- Every row is already one shared shape: `shedbuild.Row` in the recipe (`internal/shedbuild/recipe.go`) and `shedengine.ProducerDef` in the engine (`internal/shedengine/producer.go`), which carry each row's routing (`OnDone`, `OnStuck`, `OnReroute`) and budget around the `ShedProducer` seam.
- The entry gate goes there, as one more field beside the routing: an `entry_gate` key on `Row` and its counterpart on `ProducerDef`.
- Shed's loop checks the gate before it calls `ShedProducer.Call`; a false gate records the row as skipped and follows `OnDone`, so no producer implements anything, and every row of every recipe gets the gate for free.
- The `ShedProducer` interface itself stays one method, `Call`.

### The check's form (orch recommendation, not a decision)

- Reading config: shed knows no config, so the gate reads through the one told lookup task `shedrecipe-capabilities` adds, `Setting(module, key)`, filled by the recipe's owner from the pair's config stack, a task's `run_settings` included. The gate is read when the row is reached, never when the recipe loads, so a setting changed mid-run counts.
- A small fixed form, no expression language:
  - `darn.review_max_bounces`: true when the value is set and not zero, false, empty or an empty list.
  - `regression.mode != off` and `== <value>`: one comparison against a literal.
  - A list of such checks: all must hold.
  - No or, no arithmetic, no free code: a case that needs more gets a setting of its own.
- Recipe load refuses a gate naming an unknown module or key, as `shedrecipe-capabilities` refuses unknown keys.
- Examples: Darn-Review `entry_gate: darn.review_max_bounces`; Open-IDE `entry_gate: batten.open_ide`; a regression row `entry_gate: [loom.regression_mode != off]`.

### Open

- The form of a check: a config key that must be true or above 0, or a small fixed set of predicates; never free code.
- Whether a task's `run_settings` can switch a gate per task: it can today for darn's budget and batten's flags, since both read the pair's config stack.

## XDR holds stack (from note loom-gates-xdr-stacking)

### Seen (orch, 2026-10-20)

On OSL-1046, Cortex XDR holds every test binary with an unseen hash for 8-10 s before it runs (`docs/benchmarks/gate-test-xdr.md`).
A loom run pays that hold at every gate that tests a package the run just changed, and those gates stack:

- the card gate after every card, over the card's own packages (`pattern/PATTERN-test-speed.md`, Gate per tier); each card changes a package, so each card gate links at least one new binary and waits out its hold, and a single-package test gets no overlap from parallelism;
- the batch gate once per batch, `--tags integration` over the batch's packages;
- Webster's verify gate and Publish, each the plan's whole verify, 3-6 min on this machine depending on how widely imported the changed packages are, plus `publish_verify` (the tmux tier) at Publish;
- every Webster-Burler-Fix and Merge-Burler-Fix round gate, over the impacted set.

Finalize and Merge-In skip their verify when the tree is the one already verified (`pattern/PATTERN-verified-tree.md`), so the stacking is in the card, batch, round and whole-suite gates, not in repeats on an unchanged tree.
For a plan of 10-15 cards the card gates alone wait out 10-15 holds, roughly 2-3 min of nothing but XDR, before any test time.
Webster's card, batch and round gates are already scoped; only its plan-level verify gate runs the whole plan verify (`internal/loomshed/doc.go`), and Publish runs it again, which hits the cache when the parent has not moved.
So the stacking is mostly the many small gates that each link a new binary, not an unscoped Webster.

### Ideas (orch, not decided)

- Measure first: clock one ordinary loom run on OSL-1046 per gate kind, so the split between XDR holds and test time is known.
- Dropped (2026-10-20): building test binaries in the background while the next card is written.
  The build would read a working tree the next card is editing, so it can compile a half-written tree whose hashes no later gate uses, and a later card that changes a dependency changes the hashes anyway; building from the card's commit in a separate checkout costs more than it saves.

### Operator decision (2026-10-20)

Webster's plan-level verify gate keeps the whole suite: Webster implements loom's large tasks, which touch much, so it checks everything before it reports done.
Scoping that gate is not an option here; darn, for small changes, is where gates scope to the diff.

The card gate keeps its tests too: a card is atomic by design, it leaves every test passing, so a failure points at exactly the card that caused it.
Moving the tests to the batch gate would lose that, so it is not an option either.
What remains is measuring the hold's share of a few ordinary loom runs on OSL-1046 before anything is built.

### Seen (orch, 2026-10-20): first loom run on OSL-1046 after the gate fixes

`lenses-module` took 1h50m from start to landing on OSL-1046 (`lyx report timing`): discussion 26m50s, plan 42m50s, webster 35m32s, merge 22s, publish-finalize 4m06s.
The operator puts a typical loom run on Neve, which has no XDR, at about 2 hours, so this one run is in line with Neve.
One run settles nothing: compare the mean of several loom runs on each machine before drawing a conclusion.
