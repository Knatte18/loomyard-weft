# Card 14 — vocab-sweep-tests-core

**Edit:**
- `cmd/lyx/boardguard_test.go`
- `cmd/lyx/checkedcall_test.go`
- `cmd/lyx/cwdmutation_test.go`
- `cmd/lyx/destructiveguard_test.go`
- `cmd/lyx/rawgitmutation_test.go`
- `cmd/lyx/spawnobservability_test.go`
- `cmd/lyx/stencilseed_integration_test.go`
- `cmd/lyx/uncontainedwrite_test.go`
- `contracts/stencils/claims_test.go`
- `internal/batcher/config_test.go`
- `internal/battencli/arm_seed_test.go`
- `internal/battencli/cli_test.go`
- `internal/battencli/commitstatus_test.go`
- `internal/battencli/lifecycle_integration_test.go`
- `internal/battencli/records_integration_test.go`
- `internal/battencli/specfill_test.go`
- `internal/battencli/wire_test.go`
- `internal/burlercli/wiring_test.go`
- `internal/configcli/configcli_integration_test.go`
- `internal/hubgeom/parent_test.go`
- `internal/hubgeom/reedgeom_integration_test.go`
- `internal/ideengine/spawn_hub_integration_test.go`
- `tools/sandbox/main_test.go`
- `tools/sandbox/report_test.go`
- `tools/sandbox/suite_test.go`
- `tools/wordswap/swap_test.go`

**Intent:** Reword every bare `weft`/`warp` occurrence (case-insensitive substring, as the fabric-vocabulary scan matches) in the listed test files, in `cmd/lyx`, `contracts/`, `tools/` and `internal/` packages `batcher` through `ideengine`, ahead of card 18 extending the scan to `_test.go` files outside the owner set (D10).

Follow the Shared Decisions `vocab-sweep` rule:
- Owner-set exports in the `vocab-exempt` table stay as they are.
- Local identifiers, test names, comments, subtest names and string literals are reworded with the neutral vocabulary (fabric, pair, records side, code side), or rephrased so no side needs naming.
  A local helper or variable is renamed throughout its file; a renamed test function keeps its meaning.

A string a test compares against production output (an envelope key, an error message, a CLI flag or usage text owned by fabric) is not reworded here.
If it names a side, it is an owner-set spelling the test may not change.
Report it in the card's deviation note, so card 18 can add it to the `vocab-exempt` table with a reason.

Run the package tests, including `-tags integration` for any reworded integration file.

**ImpactSummary:** Test-only renames and rewording across the listed files; no production code changes.
