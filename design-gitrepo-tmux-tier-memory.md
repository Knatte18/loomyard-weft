# Investigate internal/gitrepo tmux-tier cost: 16.5s and 3 GB peak memory

`gitrepo-tmux-tier-memory` · Note · audit, fabric

## Seen (2026-10-16)

`docs/benchmarks/test-suite-weight.md`, tier 3 (`tmux`), measured serially package by package: `internal/gitrepo` took 0.31s wall and 82.5M peak memory at `df36bb2a4` (2026-10-10), and 16.50s wall, 13.82s CPU and 2992.7M peak memory at `112cccb8f` (2026-10-16, idle machine, commit `3617eae27`).
In tiers 1 and 2 the package is unchanged (under 1.3s and about 110M).
The package has no `tmux`-tagged test file: its test files are untagged or `integration`, so the tmux tier runs the same untagged tests that took 0.46s and 103.4M in tier 1 of the same run.
The spike is therefore likely a measurement artifact of that one run (a fresh build of the test binary under the new tag set, or something else on the machine), not a growing test.

## Idea

First re-run `go run ./cmd/testtiming -resources -tags tmux` for this package alone on an idle machine; if the spike does not repeat, close the note.
Otherwise find which test in `internal/gitrepo` drives the time and memory, whether the 3 GB peak is the test process or a child it spawns, and whether the test can be made lighter or belongs in another tier.
The board has no `gitrepo` label, so `fabric`, its main consumer, stands in.