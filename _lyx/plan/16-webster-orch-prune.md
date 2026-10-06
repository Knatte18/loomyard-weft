# Card 16 — webster-orch-prune

**Edit:**
- `internal/webstercli#`
- `internal/orchcli#`
- `internal/orchengine#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of `internal/webstercli`, `internal/orchcli` and `internal/orchengine`.
`internal/websterengine` is core's and stays untouched (Decision 1).

These packages hold most of the in-scope `failed` entries: in `internal/webstercli` the `TestAcceptAuditCmd_*`, `TestRestorePlanCmd_*` and `TestRebaselineCmd_*` families, `TestResetCmd_UnknownToRefusesNamingBothTargets` and `TestRunCLI_GroupGuard_OutsideGitRepo`, and in `internal/orchcli` (`TestStatus_*`, `TestCycle_WritesRequest`, `TestStop_*`, `TestWatch_ReportsAlreadyRunningWhenLockHeld`).
Each gets its Decision 8 verdict under its own name before any fold or merge touches it.
`internal/webstercli`'s `integration` files, and `acceptaudit_test.go`, `restoreplan_test.go` and `reset_test.go` in particular, are scenario-merge leads over one hub; a refusal's way-forward text (`PATTERN-refusal-way-forward`) is a unique assertion under Decision 2.
`internal/webstercli`'s `cli_tmux_test.go` stays in the `tmux` tier and never joins an `integration` scenario.
`internal/orchengine`'s candidates are fold leads over its notice and watcher checks; `PATTERN-orch-pane-single-writer` tests keep the idle gate.

**ImpactSummary:** Test files of three webster and orch packages only; no production change.

**Commit:** 16: Prune webster and orch CLI tests

**Verify:**
go test -count=1 -tags tmux ./internal/webstercli/
go vet -tags llm ./internal/webstercli/ ./internal/orchcli/
