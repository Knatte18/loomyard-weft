# Card 21 — reclassify-loom-webster-shed

**Edit:**
- `internal/loomcli/circling_integration_test.go`
- `internal/loomcli/integration_driverbootstrap_test.go`
- `internal/loomcli/landingdeps_integration_test.go`
- `internal/loomcli/parentreview_integration_test.go`
- `internal/loomcli/smoke_driverstrand_test.go`
- `internal/loomcli/smoke_helpers_test.go`
- `internal/loomcli/smoke_starttail_test.go`
- `internal/loomcli/wiring_carryover_integration_test.go`
- `internal/loomcli/wiring_commitstatus_integration_test.go`
- `internal/loomcli/wiring_publish_integration_test.go`
- `internal/loomcli/wiring_rework_integration_test.go`
- `internal/landingshed/finalize_integration_test.go`
- `internal/landingshed/publish_integration_test.go`
- `internal/webstercli/cli_integration_test.go`
- `internal/webstercli/reset_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/shedcli/parity_test.go`
- `internal/battencli/lifecycle_integration_test.go`

**Uses:**
- `plan:internal/hubforge#CopyHub`
- `plan:internal/hubforge#Shape`

**Intent:** The hub-taking tests of loom, landing, webster, shed and batten, under the classification in Shared Decisions.
Every test here seeds config and adds a pair or runs a verb against its hub, so every one copies; `newWiredPairFixture` in `smoke_helpers_test.go` seeds four modules' config before adding its pair, so it copies the `.` no-pair shape and keeps its own `AddPair`, and `webstercli`'s `seedPersistentPreRunFixture` copies the shape of the anchor it is passed.
Pure refactor of fixtures: the existing assertions cover it.

**ImpactSummary:** Fixture construction changes in eighteen test files across five packages; no production code and no assertion changes.
