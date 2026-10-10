# Card 22 — reclassify-reed-orch-shuttle

**Edit:**
- `internal/reedcli/cli_integration_test.go`
- `internal/reedcli/naming_integration_test.go`
- `internal/reedcli/smoke_coldstart_test.go`
- `internal/reedcli/smoke_debuglog_test.go`
- `internal/reedcli/smoke_dotfill_test.go`
- `internal/reedcli/smoke_lifecycle_test.go`
- `internal/reedcli/smoke_resume_test.go`
- `internal/reedcli/smoke_selvage_keepalive_test.go`
- `internal/reedcli/smoke_staterecovery_test.go`
- `internal/reedcli/smoke_switchseed_test.go`
- `internal/reedcli/smoke_teardown_test.go`
- `internal/reedcli/smoke_warmpath_test.go`
- `internal/reedcli/unlessname_integration_test.go`
- `internal/reedcli/watchdog_integration_test.go`
- `internal/reedcli/watchdogreap_integration_test.go`
- `internal/orchcli/notify_integration_test.go`
- `internal/orchcli/refusal_integration_test.go`
- `internal/orchcli/smoke_cycle_test.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/shuttlecli/smoke_guardrail_test.go`
- `internal/shuttlecli/smoke_interrupt_test.go`
- `internal/shuttlecli/smoke_run_test.go`
- `internal/shuttlecli/smoke_turns_test.go`
- `internal/seatengine/smoke_seats_test.go`
- `internal/treadleengine/smoke_judge_test.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/burlerengine/smoke_round_test.go`

**Uses:**
- `plan:internal/hubforge#CopyHub`
- `plan:internal/hubforge#Shape`

**Intent:** The hub-taking tests of reed, orch, shuttle, seat, treadle and burler, under the classification in Shared Decisions.
Every test here brings reed up, runs the binary or a verb, or seeds config, so every one copies the `.` shape its call names; `notify_integration_test.go` adds `notify-task` and reads it back by name, so it copies the no-pair shape and keeps its `AddPair`.
A hub copied for a reed-driven test keeps the hub's own `reedengine.ServerName` key, derived from the copy's path, so no two copies share a server; card 6's registration of that key, `reedengine.ServerName(h.Path)` before the first boot, follows the copy unchanged.
Pure refactor of fixtures: the existing assertions cover it.

**ImpactSummary:** Fixture construction changes in twenty-seven test files across six packages; no production code and no assertion changes.

**Verify:** lyx gate test --tags tmux ./internal/reedcli ./internal/orchcli ./internal/shuttlecli ./internal/seatengine ./internal/treadleengine ./internal/burlerengine -- -run '^$'
lyx gate test --tags llm ./internal/reedcli ./internal/orchcli ./internal/shuttlecli ./internal/seatengine ./internal/treadleengine ./internal/burlerengine -- -run '^$'
