# Build coverage maps with `lym gate test --map`, retire `gate testmap`

Building the per-package coverage maps becomes a mode of `lym gate test`, set by a `--map` flag.
The `lym gate testmap` verb goes, with its incremental planning, audit round, build lock and wait, `--max` and run-record ordering.
One way to build a map, through the gate, replaces two.

The maps keep their format, location and consumers.
`lym gate redundant` and the report step read them as before.

## What changes

- `lym gate test --map [<targets>...]` measures the tier 0 and tier 1 tests of each package in the call's scope in one pass,
  so each map holds both tiers' entries.
  The scope is chosen as for any call: targets, `--all`, `--card`, `--base`, or the diff from the merge base.
- Each package gets one `-cover` test binary built in `set` mode.
  Each mapped test runs as its own process from it, so each test gets its own block counters.
  `internal/testcover` reduces the profiles and the map is written once per package, whole.
  Spawn-marked tests are not run and keep their spawn verdict; skipped, failed, timed-out and killed tests get an entry with no blocks.
- `lym gate testmap` is an unknown command.
  The planning, audit and self-check code in `internal/gatecli`,
  the code in `internal/testcover` that only that planning reached (`testselect.go`, `declcompare.go`, `upkeep.go`),
  and `gateslot.Pool.TryAcquire` go with it.
- `lym gate redundant`'s refusals that name a map rebuild now name `lym gate test --map <package>`.
  The one for a map built from other Go files also says to commit or revert the worktree's edits first.
- `README.md`, `docs/overview.md`, the `gatecli`, `gateslot`, `testcover` and `dobbyengine` package docs, `gateslot/template.yaml`
  and the `PATTERN-test-speed`, `PATTERN-daemon-wakeups` and `PATTERN-gate-slot` backgrounds name the flag.
  The `cmd/lym/daemonwakeups_test.go` row for the removed lock wait goes.

## Design decisions

- **Flag shape.** `--map` refuses `--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `-C` and words after `--`,
  each refusal naming `lym gate test --map` with targets or a scope flag as the way forward.
  It also refuses a hub whose tier 0 or tier 1 is not a `go-json` tier, since a map exists only for Go.
  `--map` is part of the detached call's key words, so a map call never joins a plain call's run, nor a plain call a map run.
- **Whole rebuilds.** Each package is measured whole and its map replaced whole, except that a failed build keeps the old entries and records the failure.
  Packages run one after another in import-path order; the next package's coverage build may start while the current one's tests run.
- **Bounds.** Tests of a tier run side by side up to its `parallel` (with `tier_parallel` applied), tier 0 first and then tier 1.
  Each runs under the `redundancy_test_memory_mb` cap, and the count is clamped to the cgroup headroom
  and to the tier's `memory_mb` divided by that cap.
  A hub whose tier sets `memory_mb` below `redundancy_test_memory_mb` is refused, with `lym config gate` as the way forward.
  The memory guard samples the run's process tree and names a killed `.test` process.
  The coverage builds run outside the per-test cap and the tier's `memory_mb`.
- **Alone-rerun.** A test that fails, times out or is killed beside others reruns once alone, and that outcome stands.
  The report lists each test that passed only alone.
  Such a test enters the map as a plain pass, so that list is the only trace of a collision.
- **Clean-tree check.** A map is written only when HEAD is unchanged since the start
  and no file owned by the package or an in-module package it links is modified or untracked.
  The check runs after the build lists the linked packages and again before the write.
  A dirty package is listed as unmeasured and the call goes on; a moved HEAD ends the call.
  Either makes the exit non-zero.
  A gitignored Go file the build compiles is not seen by the check.
- **Slots and records.** A map run holds as many gate slots as the larger of tier 0's and tier 1's asks, a tier asking for every slot ranking highest.
  It writes a log and no run, green, verify or failure record, so `lym gate report` totals no map run and no later call skips a tier because of one.
  It exits non-zero on a failed, timed-out or killed test after the rerun, a failed build, or a clean-tree failure,
  and still writes the maps the check lets through.
- **Cost.** On a hub whose tier asks `slots: all`, a map run holds every slot for its whole length, and other gates on the hub wait.
  The run is on command only.
- **Help text.** To keep `lym gate test --help` under the help byte cap, its `--gate finalize` and `--upto 2` examples
  and the per-tier summary line sentence are dropped, and a `--map` line and example added.

## To check by hand

- `lym gate test --map <package>` from the prime and from another worktree was not run against a real hub here.
  Check that each writes its map to the shared maps and the anchor `.lym` respectively.
- The decision record leaves two design findings open:
  the clean-tree check's handling of a package whose linked-package listing fails (a failed listing is checked against every modified or untracked file of the module),
  and the run-time bounds of a map run, which are stated only as "on command" with no timeout of its own.
- A test that writes the real home directory, a repository file or another fixed path can collide with its siblings in ways the alone-rerun hides as a pass.
  The list of tests that passed only alone is how a new case shows up.
- Old per-package `.build.lock` files left beside existing maps are inert; no reader opens them.
