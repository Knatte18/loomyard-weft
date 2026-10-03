# Card 3 — claudeengine launches every session with a pointer to prompt.md

**Edit:**
- `internal/shuttleengine/claudeengine/claudeengine.go`
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/command_test.go`
- `internal/shell/shell.go`
- `internal/shell/posix.go`
- `internal/shell/pwsh.go`
- `internal/shell/shell_test.go`
- `internal/loomcli/driverprompt.go`
- `internal/loomcli/driverprompt_test.go`
- `internal/loomcli/integration_driverbootstrap_test.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/integration.go`
- `internal/websterengine/runlevel_test.go`

**Intent:**
D12: `Prepare` keeps writing `spec.Prompt` to `prompt.md`, and `buildLaunchCmd` now passes one fixed, quoted pointer argument instead of expanding the file with `sh.ReadFile`:
`Read <absolute prompt.md path> in full first; it is your complete, authoritative instructions.`
The path is made absolute before it is rendered.
`maxLaunchPromptBytes` now bounds the launch argument (the pointer), never `spec.Prompt`, so a prompt of any size launches;
the refusal text says so, and is reached only by a pathological run-directory path.

This removes a guard.
Before, `Prepare` refused any `spec.Prompt` over 30000 bytes, because the whole prompt expanded into one command-line argument and Windows caps a process command line at 32,767 characters.
After, nothing at launch bounds prompt size: the prompt is a file the session reads, and the pointer's "in full" tells it to page through a file longer than one read returns.
The launch line itself stays bounded, since the pointer is fixed text plus one path.

`Shell.ReadFile` loses its only caller:
it goes from the `Shell` interface and from `posix.go`, `pwshShell` keeps the idiom as an unexported helper for its own `Source`, `shell_test.go` drops the posix `ReadFile` case and pins pwsh's through `Source`, and `command.go`'s header comment stops naming a prompt-file read idiom.

Webster drops its own copy of the mechanism:
`MasterPromptPointer` and `masterPromptFileName` go from `integration.go`,
and `Run` puts the rendered Master prompt straight into `spec.Prompt` without writing `master.md`.

Every role's first user message changes from the prompt to the pointer;
the decision record lists this as an open risk.
The one test that reads the prompt off the launch line, `integration_driverbootstrap_test.go`'s stub driver, greps the drive-report path out of `$1`;
its stub now takes the `prompt.md` path from the pointer in `$1` and greps the report path from that file.
`driverPrompt`'s doc comment and `TestDriverPrompt_StaysWellUnderLaunchPromptCap` stop citing the launch cap:
the prompt stays short because a copied-in skill would go stale, and the test keeps its length bound under that reason.

Tests:
`settings_test.go`'s `TestPrepare_PromptLaunchLimit` becomes a prompt over 30000 bytes that launches, with the launch line carrying the pointer to `prompt.md` and not the prompt text, and `prompt.md` holding the full prompt.
`command_test.go` pins the pointer argument on the launch line for posix and pwsh.
`runlevel_test.go` pins that Master's `Spec.Prompt` is the rendered prompt and that no `master.md` is written.

**ImpactSummary:** every shuttle-launched agent (all loom, batten, burler, webster and orch roles) receives the pointer as its first message, so only a consumer reading the first message for content is affected, and the one such test is in scope; `Shell` narrows by one method with no other caller; webster loses its private pointer file.
