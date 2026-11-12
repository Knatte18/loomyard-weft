# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-12 18:40 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 2d · Owner `neve`

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

## Order (operator, 2026-11-12)

1. First the way to make and keep the maps: the upkeep pass (`lym gate testmap --upkeep`, landed) and the prime dobby that starts it (task `testmap-upkeep-dobby`, running; a running task cannot join a project).
2. Then the candidate list: `test-redundancy-candidates` builds the full set of maps and lists the redundant-test candidates per package, cutting nothing.
3. Only then the cut stage, `redundancy-sweep-existing`, from that list.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 2d<br>51m | testkit, gate |
