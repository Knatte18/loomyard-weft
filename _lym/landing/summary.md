# gate testmap: an --upkeep pass that yields its slot and writes per round

`lym gate testmap` gains the run side of the coverage-map upkeep design (D8, D9, D10).
A pass now holds a gate slot only while another stays free and gives it back when a gate arrives.
It runs tests that declare `t.Parallel()` side by side and writes the map after each round, so a yield, a signal or a landing costs only the runs in flight.
The prime dobby that starts these passes is the follow-up task `testmap-upkeep-dobby`.

## What changes

**`--upkeep` (hidden flag, `internal/gatecli/testmap.go`).**
The call registers its own process as `runproc.RoleTestMap` under `runproc.HubOwner`.
It refuses package arguments, and refuses while another live process holds that registration.
A failed registration is reported, and the call exits before it plans.
Only a `--upkeep` call writes `testcover.ExitRecord`; every other call leaves the record as it was.
Without an explicit `--max`, a `--upkeep` call has no package cap.

**Passes and HEAD.**
A call naming no package is a pass.
The call reads HEAD before each job, after each round, and on each wake of its slot wait.
Once HEAD moved, the call ends and counts the jobs it left as queued.
Dirty-skipped packages, failed-at-HEAD packages and spawn-marked tests still never count as queued.

**Slot discipline (D9).**
`gateslot.Pool.TryAcquireSpare` takes a slot only while two are free and no drain waits.
`gateslot.Pool.Contended` reports every other slot held, or a drain waiting.
A pass takes its slot through `TryAcquireSpare`, and polls `Contended` once a second while it holds the slot (`internal/gatecli/testmapyield.go`).
When a gate needs the slot, the pass kills its runs in flight and releases the slot and the package's build lock.
It then waits, polling from one second and doubling to thirty, and reads HEAD on each wake.
With a slot again it retries the lock without waiting, and skips the package as queued when the lock is held.
Holding both, it re-reads the map and plans the package afresh at the planned HEAD through `planLockedPackage`.
On a hub with fewer than two slots a pass takes no slot: it plans, writes the maps the plan moved, and counts every job queued.
A call naming packages keeps the blocking take, never yields, and waits with the same backoff for each named package's build lock.
A call inside an inherited slot neither takes a slot nor yields.

**Runs inside a job (D10, `internal/gatecli/testmappackage.go`).**
The new `testcover.TestFunc.Parallel` marks a test that calls `t.Parallel()` as a top-level statement with only `tiergate.Require` calls before it.
Such tests run side by side, up to tier0's parallel, each in its own process with its own profile; every other test is a round of its own.
After each round the job:
- takes the dirty read over every linked package;
- reads HEAD, and on a dirty file or a HEAD move discards the round and ends the job;
- otherwise merges the round into the map and writes it.

A HEAD-move skip also counts the package as queued.
The self-check now runs on the first round that yields a profile.
A failing kept entry is dropped from the map and queued again in the same job, so an interrupted job never leaves an unplaceable entry.

**Job order.**
Jobs are ordered by the summed `elapsed_seconds` of each package's launches in the retained `gateslot.RunRecord`s, largest first.
The remaining jobs follow in import path order.

## Design decisions

- The record says `internal/gateslot`'s take is unexported.
  The previous commit had already exported `Pool.TryAcquire`, so the spare rule is a new `TryAcquireSpare` beside it.
- `TestFunc.Parallel` lives in `internal/testcover`, a package outside the record's scope list.
  `ParseTestFuncs` already walks the test's AST, so detecting `t.Parallel()` there avoids a second parse in `gatecli`.
- A yield or a signal discards the whole round in flight, finished parallel runs included.
  Writes happen per round, so "the entries that finished" means the rounds that finished.
- Calls that name packages also end at their next job once HEAD moves.
  Measuring at a HEAD that has already moved would only discard every round afterwards.
- `registerUpkeepPass` lists the live records, then registers.
  Bound: two `--upkeep` calls that start in the same instant can both pass the check.
  The dobby starts at most one, so this only matters for two hand calls.
- The new waits are listed in `cmd/lym/daemonwakeups_test.go` and `PATTERN-daemon-wakeups`.

## Tests

- `internal/gateslot/pool_test.go`:
  - `TryAcquireSpare` takes a slot with two free and refuses with one free, none free, fewer than two slots, or a drain.
  - `Contended` reports a drain and every other slot held.
- `internal/testcover/testfuncs_test.go`: `Parallel` is set only for a top-level call that follows `Require` calls alone.
- `internal/gatecli/testmaporder_test.go`: run-record job order and round splitting.
- `internal/gatecli/testmap_integration_test.go` adds:
  - refusals of `--upkeep` with a package and beside a live registered pass;
  - a hand call that leaves the exit record as it was;
  - a HEAD move that discards its round, keeps the round before it, and ends the call at the next job;
  - `Meets` tests that pass only when run beside each other;
  - `TestGateTestMapPassYields`. A gate takes the last other slot, and the pass yields with its finished round kept and no entry for the killed test. A hand call then takes the yielded package's lock at once, and the pass retakes a slot once both are free.
- The existing fixtures change to fit the new rules:
  - the hubs now have two slots;
  - the test that commits lives in `zmoved`, so its HEAD move comes last;
  - `steady` no longer commits.
