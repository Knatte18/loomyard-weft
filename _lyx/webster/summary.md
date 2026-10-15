# Engine-owned config, shellguard table, provider-blind reed

All nine batches recorded `done`; none needed recovery.

- **01 settings-golden** (`629a6bf73`): golden file pinning the Claude settings document and deny notice for every deny-input combination, with `TestBuildSettings_MatchesGolden`. No deviations.
- **02 shell-guard-table** (`16eefbba4`): new leaf `internal/shellguard` with the five guard rows, rendered by `claudeengine`. Deviations: `internal/shellguard/rows.go`, `internal/shuttleengine/claudeengine/settings.go`. The fork also trimmed 211 bytes from five existing `PATTERN.md` entries to fit the new entry under `lint-pattern`'s headroom.
- **03 denied-forms-from-table** (cards 3, 4, 5, `cf3bf2d05`): the no-denied-recovery scan reads the table, `configengine.Spec` gains retired keys, and `claude.yaml` is declared in `claudeengine` and registered through `shuttleengines`. Deviations: `internal/configengine/layers.go`, `internal/configreg/registration_test.go`, `internal/shuttleengine/claudeengine/config_test.go`.
- **06 engine-loads-claude-config** (`070b4d308`): the engine loads `claude.yaml` from told `Dirs`, and `shuttleengines.New` takes them. Deviations: `internal/loomcli/smoke_helpers_test.go`, `claudeengine/config_test.go`, `claudeengine/prepare_test.go`, `claudeengine/settings.go`, `internal/shuttleengines/shuttleengines_test.go`.
- **07 shuttle-drops-claude-keys** (`b8a7d2956`): `shuttle.yaml` loses the Claude keys, retires them and renames `deny_python`. Deviations: three `internal/loomcli` test files and `claudeengine/settings.go`.
- **08 config-key-move** (`eadb370f5`): the shuttle-to-claude key move in reconcile and run settings. Deviation: `internal/configsync/movekeys.go`. The fork reported appending one test with a shell heredoc, against the edit rule; `record-batch` raised no warning.
- **09 engine-scrub-keys** (`31c3d6faf`): engines declare the environment keys reed scrubs. Deviations: `cmd/lyx/tierpurity_test.go`, `internal/reedengine/ensuresession_integration_test.go`, `internal/reedengine/env.go`, `env_test.go`, `envscrub.go`, `internal/shuttleengines/shuttleengines.go`.
- **10 strand-provider-tag** (`504c85a3b`): strands carry an opaque provider tag and the watchdog picks a namer per tag. Deviations: `internal/reedengine/namerepair_test.go`, `internal/shuttleengines/shuttleengines_test.go`.
- **11 config-package-docs** (`edf2df9e9`): `configreg` and `configsync` header docs. No deviations.

The scope guard's deviations are informational throughout, and no `record-batch` call returned warnings.
The `tmux` and `llm` test tiers were not run by the forks; the plan-level verify gate covers them.
