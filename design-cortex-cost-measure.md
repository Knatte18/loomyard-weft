# Measure what Cortex XDR costs the test suites on OSL-1046

`cortex-cost-measure` · Note · research, gate, fabric

## Seen (operator, 2026-11-09)

Cortex XDR on OSL-1046 holds each new binary for about 10 s and scans each file a test creates, and fabricengine tests create many.
The shard report (`docs/research/speed/fabricengine-shards-2026-11-09/report.md`) put the 242 s one-process wall down to load, but the 2026-11-08 figure of 37 to 41 s was measured on another machine, so the gap is machine and Cortex, not load.
The package used 1.23 cores of 16 and `pmd` and `piud` were busy in nearly every sample.

## Idea

Separate the Cortex cost from the rest: the time a freshly built test binary waits before it runs, the time file creation takes with and without a scanned folder, and the fabricengine tier 1 wall with the repository and temp folders excluded from the scan, if the operator can have that changed.
The result says whether to cut tests, cut file and binary creation, or ask for an exclusion.
