# Bench tier 1 with {test_targets} switched on

Adds `docs/research/speed/test-targets-bench-2026-11-07.md`, the measurement of tier 1 after the hub's tier commands began running `go test` over `{test_targets}`.
No code changes.

## Findings

- A whole tier 1 pass took 63.9 s against the re-measure's 65.4 s and the expected 48 s.
- `internal/fabricengine` alone took 43.9 s package time against 41.5 s before the switch.
- The whole pass started 67 test binaries, all of them packages that hold a tier 1 test.
- A static check found no tier 1 test lost by narrowing; the unnarrowed pass was not run to compare counts.

## Design decisions

- The specified package pass was carried from the whole pass's green record and ran nothing,
  because both passes used the same args after `--` over an unchanged tree.
  The parent recorded one extra pass with `-run .` as a decision, and the report uses that pass as the package measurement.
  The report states the carried run and why.
- The report records that its 67 started binaries and 199 test-holding directories differ from the brief's 66 and 196, and leaves the gap unexplained.
