# testmap: run upkeep passes under a yielding slot, per round

`lym gate testmap` gains a hidden `--upkeep` flag that makes the call an upkeep pass.
A pass drains the coverage-map backlog on its own, takes a gate slot only while another stays free, and gives it back when a gate needs it.
It runs parallel-declared tests side by side and writes the map after each round of runs.
This is the verb side of the upkeep design; the dobby that starts passes lands separately.

## What changes

- **`--upkeep` pass** (`internal/gatecli`).
  It refuses package arguments, registers its own process as `runproc.RoleTestMap` under the hub owner, and refuses to start beside a live pass.
  When its registration fails it reports that and exits before planning.
  Only a call carrying `--upkeep` reads and writes the exit record; every other call leaves it as it was.
- **Pass lifetime.**
  A pass plans once at `H` and works its queue.
  It reads HEAD before each package job and on each wake of its slot wait, and ends when HEAD moved.
  Queued work leaves out dirty-skipped packages, packages whose recorded failure names the current HEAD, and spawn-marked tests,
  so a dirty prime or a broken commit does not restart passes in a loop.
- **Slot take and yield** (`internal/gateslot`, `internal/gatecli/testmapyield.go`).
  `Pool.TryAcquireSpare` takes one slot without waiting, only when at least two slots are free and no multi-slot acquire drains.
  `Pool.Contended` reports when every other slot is held or a drain waits.
  While holding the slot a pass checks that once a second; on contention it kills its runs,
  writes the entries that finished, and releases the slot and the build lock of the package it interrupted.
  It then waits for a slot again, polling from one second and doubling to thirty seconds.
  On retake it tries the lock without waiting, skips the package when the lock is held,
  and otherwise re-reads the map and plans afresh at `H` before running another test.
  On a hub with fewer than two slots a pass takes none: it plans, re-stamps and ends.
- **Hand calls** naming packages keep the blocking slot take, never yield, wait for a package's build lock where a pass holds it,
  and rebuild the named packages whole.
  A call running under a slot inherited through `gateslot.InheritEnv` neither takes a slot nor yields.
  A call naming no package and lacking `--upkeep` runs as a pass but leaves the exit record alone.
- **Runs inside a job.**
  A job builds the coverage binary once at `H` and runs each queued test alone in its own process with its own profile path.
  Tests that call `t.Parallel()` as a top-level statement, before any other statement but `tiergate.Require`,
  run side by side up to the slot's tier-0 parallel; the others run one at a time.
  `testcover.TestFunc` gains a `Parallel` field, set by AST scan in `ParseTestFuncs`.
- **Per-round writes.**
  The map is written after each round of finished runs.
  After each round the job reads HEAD and takes the cross-package dirty read over every linked package;
  when HEAD moved or a linked file is dirty it discards that round, writes nothing more for the package, and ends the job.
  A terminating signal kills the runs in flight and writes the rounds that finished.
- **Job order.**
  Packages are taken by the summed per-package elapsed seconds of the retained gate run records, largest first,
  then every other queued package by import path.
- `PATTERN-daemon-wakeups` lists the new waits, and `cmd/lym/daemonwakeups_test.go` registers `testmapyield.go` as a file with an unbounded wait.

## Design decisions to know

- A pass yields instead of finishing its job, so a gate waits at most one poll behind upkeep, where before it could wait out a whole refresh.
  A hand call's wait for a build lock ends within one pass job for the same reason.
- A pass works one HEAD's plan, so a landing ends it at its next job or wake.
  A call without `--upkeep` and naming no package is not registered, so it can run beside a pass;
  that wastes work, and the build lock keeps the two off the same package.
- Side-by-side runs are separate processes, each with its own profile.
  A test that declares `t.Parallel()` already tolerates its package's other parallel tests in one process, so separate processes are no riskier.
- On a hub with `slots: 2` a pass runs only with no gate running; with `slots: 3` it runs beside one gate.

## To check by hand

- Run `lym gate testmap --upkeep` on a hub with three slots, start a gate meanwhile, and confirm the pass releases its slot within about a poll
  and keeps the entries it finished.
- Confirm a hand call naming a package a pass is measuring waits for the package's lock, does not yield, and rebuilds the package whole.
