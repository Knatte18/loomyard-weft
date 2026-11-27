# Measure orchcli and orchengine test time; no cut needed

The tests of `internal/orchcli` and `internal/orchengine` already run in a few seconds each,
well under the 15 seconds that would justify cutting them,
so this change edits no test and records the measured times as its result.

## Measured times

Measured on this machine with `-count=1`:

| Package | Tier | Tests | Package test time | Run wall time |
|---|---|---|---|---|
| `internal/orchcli` | tier1 | 55 passed | 1.99s | 9.1s, build included |
| `internal/orchengine` | tier1 | none | 0s | no test binary started |
| `internal/orchengine` | tier0 | 425 passed | 1.48s | 1.9s |

Every test of `internal/orchengine` is a tier0 test, so the package has no tier1 time.
The slowest `internal/orchcli` tier1 test, `TestHubOrchs_ListsWiredBeamsAndSkipsUnwiredPairs`, takes 0.86s;
the rest take 0.4s or less each.
Most of the `internal/orchcli` run's wall time is the build, which deleting tests would not shorten.

## Design decisions

- Neither package reaches 15 seconds, so the tests stay as they are:
  deleting or merging tests for a saving of about a second would cost coverage and save no meaningful gate time.
- Net test-line change: 0.

## Merge danger

**Door:** two-way
**Blast radius:** none
