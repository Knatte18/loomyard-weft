# P6 Coverage maps: build them, keep them fresh, and cut the test suite with them

Rendered 2026-11-09 20:46 UTC; every age below is as of then.
Under: [Board](README.md).

[`test-suite-trim`](design-test-suite-trim.md) · Age 0m · Owner `neve`

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
