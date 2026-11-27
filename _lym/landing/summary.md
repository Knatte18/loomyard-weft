# Measure shuttle and seat engine test times; none is slow enough to cut

This change makes no code or test change.
The aim was to cut the tier1 test time of `internal/shuttleengine`, its `claudeengine` and `codexengine` subpackages, and `internal/seatengine` at least in half.
A package qualified for cutting only when its measured time reached 15 s, in tier1, or in tier0 when it has no tier1 tests.
None does, so every test stays.

## Measurements

Measured on this machine at base `2326ef4da`, with `-count=1`.
Package times are those `go test` reports per package.

| Package | Tier1 tests | Tier1 time | Tier0 time |
|---|---|---|---|
| `internal/shuttleengine` | none | — | 1.806 s |
| `internal/shuttleengine/claudeengine` | 10 | 5.055 s | 1.957 s |
| `internal/shuttleengine/codexengine` | 7 | 0.153 s | 0.716 s |
| `internal/seatengine` | none | — | 0.809 s |

The tier1 run over the four packages took 15.1 s wall, most of it build, and started two test binaries.
The tier0 run took 2.9 s wall.
Before and after are the same figures, and the net test-line change is zero.

## Design decisions

- The 15 s threshold came after the aim was written:
  the package list came from an older coverage survey, whose times no longer hold.
  The threshold is not yet in the written scope; it is an unrecorded parent correction.
- `internal/shuttleengine` and `internal/seatengine` hold no tier1 tests, so tier1 has no time of theirs to halve.
  Their tier0 times are reported instead.

## Merge danger

**Door:** two-way
**Blast radius:** none
