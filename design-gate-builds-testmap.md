# Build the coverage maps as a mode of the gate run, one process per test in parallel

`gate-builds-testmap` · Task · sketch, gate, testkit · running · Raddle-Write

Decision record: [Sketch](decision-gate-builds-testmap.md)

## Seen (operator, 2026-11-17)

The coverage maps are worth having, but the way they are built is too complicated and ad hoc.
`lym gate testmap` is a verb of its own beside the gate, with its own planning, audit round, locks, slot handling and package lists
(`internal/gatecli/testmap*.go`, `internal/testcover`).
The survey `redundancy-confirmed-survey` (`53d40bd0`) and the payoff report behind `testmap-on-command` (`72b26d1a`)
found little use of the maps so far, but the operator holds that they can be used once building them is simple.

The core is already simple and sound:
a test binary built with `-cover` counts every block, and nothing is added to production code or to a normal build;
one process per test, each writing its own `-test.coverprofile`, gives per-test coverage;
`internal/testcover` merges the files into a map.
One process per test is needed because tests in one process share the counters, and parallel tests mix them.

## Idea

- The map is a mode of the gate's own tier 0 and tier 1 run, set by a flag on `lym gate test`, and the `lym gate testmap` verb goes.
- In map mode the gate builds each package's test binary with `-cover`, runs each test as its own process from that binary,
  one profile file per test, and `internal/testcover` merges the files into the package's map when the package ends.
- The per-test processes run in parallel across the machine's cores, bounded like the gate's own parallelism.
  Separate processes keep separate counters, so parallel runs never mix coverage.
- Risks to settle in the design:
  - Tests of one package that share state outside the process (a fixed path, a port, a tmux socket) may collide
    when run as parallel processes, though they pass run in sequence.
  - Many test processes at once need the memory guard and the per-run memory cap to cover them.
  - A `-cover` binary does not share the build cache with the normal gate build, so a map run costs a build of its own.
- What the maps are used for (test selection, redundancy, showing a reviewer the tests that cover a diff) is decided apart from this note;
  this note only makes them simple and cheap to build.
