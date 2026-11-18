# Estimate how much smaller a clean rewrite of the most patched modules would be

`rewrite-shrink-estimate` · Task · research, landing · running · Record-Write

## Why

The operator weighs a clean-room rewrite of the most patched modules (note `clean-room-module-rewrite`).
The gain sought is less code and fewer test lines, so each run reads less, the gate runs faster and reviews shrink.
This study estimates that gain before any rewrite starts.

## Scope, exact

- Modules: `loomcli`, `fabricengine`, `landingshed`, `shuttleengine`, `battencli`, `websterengine`, `driftengine` under `internal/`.
- No agent reads all of them. Only `landingshed`'s code is read in full, once; its tests are sampled, not read whole.

## Steps

1. Mechanical, all seven modules: count lines of code, comment and blank lines, and test lines per module.
   Run one duplicate-code detector (for example `dupl` at a threshold of 50 tokens) once per module over code and over tests;
   record duplicated lines and the largest clone groups. Fetch the tool with `go run` at a pinned version; install nothing globally.
2. `landingshed` only: write a spec of what the module must do and why: its job, its invariants, and each special case with its reason,
   drawing reasons from its fix-commit history, never from how the current code does it.
3. From that spec, estimate the size of a clean version: list the parts a clean design needs with a line estimate each,
   and estimate the test lines a spec-driven suite needs, by behaviour.
   Name what in today's code a clean version would drop (duplication, dead paths, workaround layers, over-long comments).
4. Extrapolate to the other six modules only from step 1's numbers and the landingshed ratio, and say plainly that it is an extrapolation.

## Output

A report under `docs/research/rewrite/shrink-estimate-<date>/` with:
the per-module table from step 1; the landingshed spec as its own file (reusable as step 1 of a pilot);
the clean-size estimate with today's size beside it; and the extrapolated range for the seven modules.
No code changes. No test runs beyond what step 1's tools need.
