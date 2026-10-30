# A seed says only that a run starts at its start, with no producer name

`seed-starts-at-start` · Note · enhancement, loom, pawl, shed

## Seen (operator, 2026-10-30)

A new beam stopped at once because the shared seed set its status file's `current_producer` to `Preflight`, a row the pawl recipe lacked.
The seed names the producer an inner run starts at, so each recipe's first row must match a name hard-coded in `loomshed.Seed` (`internal/loomshed/seed.go`), which the bootstrap shared by every inner recipe calls.

## Design (operator, 2026-10-30)

- The seed names no producer: it says only that the run starts at its start, and the recipe's first row is the start.
- A seed can then never name a row its recipe lacks, whatever recipe a run is seeded for.
- The change reaches the shared seeding for loom, darn and pawl, how the status file records the current producer before the first pass, and the verbs that move a run (`goto`, resume), which the task checks.
