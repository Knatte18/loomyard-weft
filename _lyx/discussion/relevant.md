# Relevant files

- `internal/shuttlecli/open.go` `openCmd` — the `open` verb that gains the flag and fills `OpenSpec`
- `internal/shuttleengine/open.go` `Runner.Open` — passes `OpenSpec` to the engine's `OpenLaunch` and adds the strand
- `internal/shuttleengine/engine.go` `OpenSpec`, `SessionOpener` — the spec that gains `PermissionMode`
- `internal/shuttleengine/claudeengine/command.go` `validatePermissionMode`, `buildOpenCmd`, `Claude.OpenLaunch`, `buildResumeCmd` — the existing mode rule and the open line that must carry it
- `internal/shuttleengine/spec.go` `Spec` — `PermissionMode`'s documented engine-vocabulary contract the open spec mirrors
- `internal/shuttleengine/doc.go` — package doc stating the caller-owned engine vocabulary split
- `internal/shuttleengine/claudeengine/doc.go` — open-launch paragraph to update
- `internal/shuttleengine/claudeengine/command_test.go` `TestOpenLaunch` — the table the new rows join
- `internal/shuttlecli/cli_test.go` `TestRunCLI_Run_FlagValidation`, `TestOpenCmd_ModelAliasEngine` — existing `open` CLI tests
- `docs/overview.md` — shuttle row naming `lyx shuttle open`
