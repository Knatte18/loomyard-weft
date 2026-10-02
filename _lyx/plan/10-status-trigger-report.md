# Card 10 — status-trigger-report

**Edit:**
- `internal/orchcli/status.go`
- `internal/orchcli/cli_test.go`

**Uses:**
- `plan:internal/orchengine#Config.SoftThreshold`
- `internal/orchengine/state.go`

**Intent:**
`lyx orch status` reports the soft trigger alongside the hard one.
`statusFields` adds `soft_threshold_tokens` (`cfg.SoftThreshold()`), `cycle_trigger` (`State.CycleTrigger`, empty before the first cycle) and `last_deferral` (`State.LastDeferral` as RFC 3339 UTC, or null when zero).
`threshold_tokens` keeps its name and meaning as the hard cap.
Update the verb's `Long` text to name the soft threshold, the trigger and the last deferral.

Tests: a populated state reports the trigger, the RFC 3339 deferral and both thresholds;
a zero deferral reports null.

**ImpactSummary:** Three added envelope fields on `orch status`; existing fields keep their names and values.
