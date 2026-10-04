# Card 16 — vocab-sweep-tests-rest

**Edit:**
- `internal/landingshed/finalize_integration_test.go`
- `internal/landingshed/publish_integration_test.go`
- `internal/orchcli/prime_integration_test.go`
- `internal/preflight/preflight_integration_test.go`
- `internal/preflightshed/findings_test.go`
- `internal/preflightshed/preflight_integration_test.go`
- `internal/reedcli/smoke_test.go`
- `internal/shedcli/cli_test.go`
- `internal/shedcli/parity_test.go`
- `internal/shedcli/seed_test.go`
- `internal/webstercli/cli_test.go`
- `internal/webstercli/sync_integration_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/wiring_test.go`
- `internal/websterengine/audit_test.go`
- `internal/websterengine/config_test.go`

**Intent:** Reword every bare `weft`/`warp` occurrence (case-insensitive substring, as the fabric-vocabulary scan matches) in the listed test files, in `internal/` packages `landingshed` and `orchcli` through `websterengine`, ahead of card 18 extending the scan to `_test.go` files outside the owner set (D10).

Follow the Shared Decisions `vocab-sweep` rule:
- Owner-set exports in the `vocab-exempt` table stay as they are.
- Local identifiers, test names, comments, subtest names and string literals are reworded with the neutral vocabulary (fabric, pair, records side, code side), or rephrased so no side needs naming.
  A local helper or variable is renamed throughout its file; a renamed test function keeps its meaning.

A string a test compares against production output (an envelope key, an error message, a CLI flag or usage text owned by fabric) is not reworded here.
If it names a side, it is an owner-set spelling the test may not change.
Report it in the card's deviation note, so card 18 can add it to the `vocab-exempt` table with a reason.

Run the package tests, including `-tags integration` for any reworded integration file.

**ImpactSummary:** Test-only renames and rewording across the listed files; no production code changes.
