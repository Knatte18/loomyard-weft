# Card 15 — vocab-sweep-tests-loom

**Edit:**
- `internal/loomcli/arm_seed_test.go`
- `internal/loomcli/cli_test.go`
- `internal/loomcli/commitrecords_integration_test.go`
- `internal/loomcli/driverresume_test.go`
- `internal/loomcli/friction_test.go`
- `internal/loomcli/halt_test.go`
- `internal/loomcli/landingdeps_test.go`
- `internal/loomcli/landingdeps_verify_test.go`
- `internal/loomcli/parentreview_integration_test.go`
- `internal/loomcli/parentreview_wiring_test.go`
- `internal/loomcli/sharedbootstrap_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/start_driver_test.go`
- `internal/loomcli/step_test.go`
- `internal/loomcli/wiring_commitstatus_integration_test.go`
- `internal/loomcli/wiring_commitstatus_test.go`
- `internal/loomcli/wiring_parentbranch_integration_test.go`
- `internal/loomcli/wiring_rework_integration_test.go`
- `internal/loomcli/wiring_seams_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomrecipe/overlay_seam_guard_test.go`
- `internal/loomshed/discussionwrite_test.go`
- `internal/loomshed/planwrite_test.go`

**Intent:** Reword every bare `weft`/`warp` occurrence (case-insensitive substring, as the fabric-vocabulary scan matches) in the listed test files, in `internal/loomcli`, `internal/loomrecipe` and `internal/loomshed`, ahead of card 18 extending the scan to `_test.go` files outside the owner set (D10).

Follow the Shared Decisions `vocab-sweep` rule:
- Owner-set exports in the `vocab-exempt` table stay as they are.
- Local identifiers, test names, comments, subtest names and string literals are reworded with the neutral vocabulary (fabric, pair, records side, code side), or rephrased so no side needs naming.
  A local helper or variable is renamed throughout its file; a renamed test function keeps its meaning.

A string a test compares against production output (an envelope key, an error message, a CLI flag or usage text owned by fabric) is not reworded here.
If it names a side, it is an owner-set spelling the test may not change.
Report it in the card's deviation note, so card 18 can add it to the `vocab-exempt` table with a reason.

Run the package tests, including `-tags integration` for any reworded integration file.

**ImpactSummary:** Test-only renames and rewording across the listed files; no production code changes.
