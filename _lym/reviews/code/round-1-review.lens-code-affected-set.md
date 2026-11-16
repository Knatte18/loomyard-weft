```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I checked what each change in the diff forces outside the lines it touches.

**Per change, the set found and whether it is covered**
- **Creel command `remote-control`** (`creelengine/command.go`, `creelcli/delivery.go`): the validate message, `creelengine/doc.go:416`, `creelcli/doc.go:158`, the `sessionCommands` interface (its production implementer is `*shuttleengine.Runner`, which gains `TurnOnRemoteControl`) and the test fake `recordingCommands` are all updated. Covered. `docs/research/.../report.md:351` still lists the old command set, but it is a dated audit report and so exempt. The orch watcher's own command literals (`orchengine/watcher.go:53`, `orchengine/doc.go:395`) do not need the new command. Covered.
- **`buildLaunchCmd` and `buildResumeCmd` signatures**: their only caller (`claudeengine.go:271-272`) and the tests `TestBuildLaunchCmd`, `TestBuildResumeCmd` and `TestBuildResumeCmd_NoticePosix` are updated. Covered.
- **`RemoteController` capability and `Runner.TurnOnRemoteControl`**: the shuttleengine doc and claudeengine doc are updated, and a compile-time assertion is in place. Codex has no implementation, as intended. Covered.
- **`Spec.RemoteControlName`**: only `orchSpec` sets it. The other `StartSession` callers (`loomshed/samplersession.go`, `landingshed/prsession.go`, the envkit fakes) leave it empty, as intended. Covered.
- **`orchroute.FixedStrandName` extracted from `StrandOf`**: the callers of `StrandOf` (`orchcli/notify.go:91,196,204`, `orchroute/walk.go:101`, `battencli/notices.go:64`, `tendercli/deps.go:429,460`) see unchanged behaviour. Covered.
- **New `orchengine.Paths` fields**: `orchroute.Paths` and the two test path builders (`orchcli/cli_test.go`, `orchengine/state_test.go`) are updated. The lock sits beside the file under `.lym`, the same as `state.json.lock`. PATTERN-lym-files' tracked-file lock rule covers `_lym` only, so it does not apply. Covered.
- **orch.yaml keys `remote_control` and `remote_control_name`**: `template.yaml`, `ConfigSpec` categories, `orchengine/doc.go` and `LoadConfig` are updated. Nothing else documents the orch.yaml keys. Covered.
- **orch cobra group gains hidden `rc-bar` and `rc-on`**: `PersistentPreRunE` skips the wiring for `rc-bar`, and `orchcli/doc.go` is updated. The group's own help text is left stale (finding 1). Not fully covered.
- **reed bar line 3, bindings, `bindingLinesLocked` and the new options**: `bar.go`, `reedengine/doc.go:191`, the `bindings.go` header, `pinBindingsLocked` docs, and the tests in `bar_test`, `bindings_test`, `windowsize_test`, `attach_test` and `bindings_integration_test` (whose recorder script now skips `orch`) are updated. Nothing still names `switchBindingCommandsLocked`. The slackcli help (`slackcli/cli.go:149`, `toggle.go:33`) still says "slack and away toggles", which remains true. Covered.
- **Repository rules**:
  - PATTERN-reed-panes: the click reaches the orch pane only through creel's Delivery seam and shuttle's guarded send. Covered.
  - PATTERN-child-processes: the pinned lym lines resolve through `selfexec.ResolveLymBinary`, the same way as the Slack lines. Covered.
  - PATTERN-agent-addressing: the program name `REMOTECONTROL` is checked by `NewProgramClient`'s grammar. Covered.
  - PATTERN-tolerant-records: `RemoteControlOn` is a read-only decode that does not refuse unknown keys, and the write goes through `internal/state`. Covered.
- **Commit messages and departures**: there is no `Pattern-Departure:` trailer in the range, and the directive declares no PATTERN changes. Nothing to check.

**Findings**

1. **LOW, consistency**, `internal/orchcli/cli.go:148`. The orch group's help still says "Every verb runs from the hub's prime worktree or from a beam." The new hidden `rc-bar` verb skips that wiring and runs from any session's directory, a task pair's included (`internal/orchcli/cli.go:122`; `internal/orchcli/remotecontrol.go:211-213`). The group text was not amended to note the exception.

Tool calls:
1. Grep for stale references to the renamed binding function, the old creel command list and the toggle wording. My context had only the diff, not the rest of the repo.
2. Grep for orch.yaml key documentation and program-name registries in pattern/ and docs. These had not been read.
3. Grep for verb registries in cmd, cliwire and clihelp, and for program rules in PATTERN-agent-addressing. That file had not been read.
4. Grep for the orch group's "Every verb runs" claim and for other mentions of the new verbs and program. I needed the group help text outside the diff hunk.
5. Grep for other line-3 layout docs and for `StrandOf` callers. These callers were outside the manifest.
6. Grep for other listings of creel command names. I needed this to complete the creel-command set.
