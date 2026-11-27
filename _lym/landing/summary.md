# Memory guard: a slice other load filled is an infra stop, not a failure

When the user slice's memory limit is nearly spent, the gate's memory guard kills the largest test binary of 1 GB or more.
Until now it called every such kill a failure of that package's tests.
It never reran the tier, and its way forward pointed at the code under test.
In the kills that issues #267 and #270 report, the killed `cmd/lym` or `internal/gatecli` binary held 1 to 2 GB of a 16 GB slice.
Some other load held the rest, so a comment-only change got a red tier1 that no fixer could clear.

## What changes

- **The guard tells slice pressure apart from its own run's growth.**
  On a pressure kill, `memoryGuard.kills` sums the resident memory of the whole tree it guards.
  When that sum is under half of the limit's use (`pressureOwnShareDivisor`), the kill is a *slice-pressure kill*.
  A kill over `gate.memory_quit_mb`, and a pressure kill where the run itself holds most of the limit's use, keep their old line and handling.
- **The kill line names the other holders and gives a way forward that fits.**
  A slice-pressure kill's line states the run's own share of the limit's use.
  It names up to three of the largest other processes in the limit's cgroup, by executable, pid and resident MB.
  It calls the kill an infrastructure stop, not a test failure, with the way forward to wait for that load to end and re-run.
  The new `proc.ResidentInCgroup` lists a cgroup's processes with their resident memory.
  It returns `errors.ErrUnsupported` on Windows.
- **The tier waits and reruns once.**
  Some tier runs end with slice-pressure kills as their only guard stops and no failing test.
  `retryTier` reruns such a tier once after the existing rerun delay, the same path a resource shortage or a runner crash takes.
  When the rerun ends the same way, the run carries `pressureStop`:
  - its closing-line label calls it an infrastructure stop;
  - it stops no `--stop-on-fail` call;
  - it does not count as a failure that judged the code.
- **The call says no test judged the code.**
  Every failed run may be a pressure stop or a runner crash, with at least one pressure stop.
  The call then closes with the new `gateslot.MemoryPressureLine` in place of `gateslot.RunnerCrashLine`.
  `gateslot.FindMemoryPressureStop` is its finder.
  A card gate's detached runner hands such a verdict to no cold fixer, just as it handles a runner crash.
- A `--copies` run is a slice-pressure stop when at least one copy was one and every other copy passed or was one too.

The package docs of `gatecli`, `gateslot` and `proc` describe the new classification, the rerun and the line.

## Design decisions

- The guard still kills under slice pressure.
  Holding back would leave the kernel to pick a victim at `memory.max`, which may be an agent session.
  What changes is how the kill is reported and that the tier gets another run.
- The test for "most of the slice's use lies elsewhere" is the guarded tree's resident total against half of the limit's use.
  The tree counts every process the guarded tree started, compilers and nested binaries included, so a tier whose own parallel test binaries fill the slice still fails as before.
- The rerun is one rerun after `rerunDelay`, the bound the shortage and crash reruns already use, not an open-ended wait.
- `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` (`cmd/lym/testonlycode_linux_test.go`) was checked by reading it, not by measuring it.
  It loads the whole module with `packages.LoadAllSyntax`, builds SSA for every package and runs RTA over it.
  That memory is large but bounded by the module's size, so it is not a test that holds memory without bound.
  The board note `whole-repo-guards-off-gates` already covers moving such whole-repo guards off the task gates.

## Bounds

- The verify sites built on `internal/verifytree` do not read `gateslot.MemoryPressureLine`.
  They see a failed run whose failure report and closing line call it an infrastructure stop, and they may still start a fixer for it.
  Those sites are the loom verify, the landing gates and `lym webster verify`.
- The other holders are read once, at the kill, so a short-lived load that has ended by then is not named.

## Tests

- `TestMemoryGuard` has three new rows.
  A short limit that other load fills kills the largest tree as a slice-pressure kill, and its full line, other holders included, is pinned as a literal.
  A cgroup that cannot be read gives the "could not be read" clause.
  In the row for a short limit the run fills itself, the run's own share is raised so it stays an ordinary pressure kill.
- `TestRunTierMemoryGuard` covers the tier run's wiring: a slice-pressure kill marks the result for a rerun, and a kill over the cap does not.
- `TestRetryTier` checks four cases:
  - a pressure stop followed by a pass;
  - a pressure stop followed by an ordinary failure;
  - two pressure stops, which carry `pressureStop`;
  - a shortage followed by a pressure stop, and a timed-out pressure stop, neither of which carries one.
- `TestFailedRunLabel` pins the new label.
  The gateslot line tests pin the pressure line's finder against the crash line both ways.
  The card fixer test pins that a pressure line spawns no fixer.
- `TestProcessTreeReads` checks that `ResidentInCgroup` lists the test process and its child in their own cgroup and its parent, and nothing in an empty child cgroup.

## Merge danger

**Door:** two-way
**Blast radius:** gate
