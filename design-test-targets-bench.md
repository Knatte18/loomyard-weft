# Bench tier 1 with {test_targets} switched on

`test-targets-bench` · Task · research, gate, testkit · running · Open-Window

## Why (operator, 2026-11-07)

The hub's tier0 to tier2 commands now run `go test` over `{test_targets}` (Bolt config commit of 2026-11-07), so `go test` starts only the test binaries that hold a test of the tier.
`docs/research/speed/fabric-test-speed-remeasure-2026-11-07.md` measured tier 1 before that switch: 65.4 s for a whole pass, 41.5 s for `internal/fabricengine` alone, with neve under load.
The gate landing (`3aca2e50`) expected about 48 s for a whole pass with the placeholder.

## Runs (exact cap)

Run each of these once, in the foreground with the Bash tool timeout 3600000, and no other run:

1. `lym gate test --tier 1 --all -- -count=1 -shuffle=off`
2. `lym gate test --tier 1 ./internal/fabricengine -- -count=1 -shuffle=off`

`-shuffle=off` is go test's default order and defeats the gate's green-record carry over an unchanged tree, as the re-measure found.
Before each pass record the one-minute load from `/proc/loadavg` and the runs in flight from `lym loom peers`.

## Report

Write `docs/research/speed/test-targets-bench-2026-11-07.md`: both wall times beside the re-measure's figures and the 48 s expectation, the number of test binaries started against the unnarrowed pass (196, of which 66 hold a tier 1 test), the load and runs in flight, and whether the narrowed pass lost any test.
Commit the report and no code.
