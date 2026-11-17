# Build the coverage maps through `lym gate test --map`

All three batches reached `done`; the final head is `c785978b`.

## Batch 01 — gate-test-map-mode (head `74fc052f`)

`lym gate test --map` now runs the map measure loop over the scope's packages, and the `testmap` verb with its planning, audit, locks, wait and ordering is retired.
Deviations the scope guard listed: `cmd/lym/testonlycode_linux_test.go`, `internal/gatecli/testmap.go`, `internal/gatecli/testmap_integration_test.go`, `internal/gatecli/testmappackage.go`.
Departures: none recorded in the digest.

## Batch 02 — testcover-planning-delete and map-run-tier-bounds (head `c1b4d92a`)

The testcover selector and upkeep planning were deleted.
The map run now respects each tier's parallel and memory bounds, reports tests that passed only alone, holds the tiers' slot ask, builds no lym, and exits non-zero on a failed measurement.
Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/gatecli/testmap_integration_test.go`.
Departures: none recorded in the digest.

## Batch 04 — map-run-clean-checks and redundant-map-way-forward (head `c785978b`)

A map is written only when HEAD is still and the package's and linked packages' files are clean; the rest are listed as unmeasured.
`lym gate redundant`'s map refusals now name `lym gate test --map` as the way forward.
Deviations: `internal/gatecli/testmap.go`, `internal/gatecli/testmap_integration_test.go`.
Departures: none recorded in the digest.

## Scope guard

Every digest reported `status: done` with no renames and no warnings.
The listed deviations are informational only.
