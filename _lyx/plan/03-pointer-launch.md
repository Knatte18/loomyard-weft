# Card 3 — claudeengine launches every session with a pointer to prompt.md

**Edit:**
- `internal/shuttleengine/claudeengine/claudeengine.go`
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/command_test.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/integration.go`
- `internal/websterengine/runlevel_test.go`

**Intent:**
D12: `Prepare` keeps writing `spec.Prompt` to `prompt.md`, and `buildLaunchCmd` now passes one fixed, quoted pointer argument instead of expanding the file with `sh.ReadFile`:
`Read <absolute prompt.md path> in full first; it is your complete, authoritative instructions.`
The path is made absolute before it is rendered.
`maxLaunchPromptBytes` now bounds the launch argument (the pointer), never `spec.Prompt`, so a prompt of any size launches;
the refusal text says so, and is reached only by a pathological run-directory path.

Webster drops its own copy of the mechanism:
`MasterPromptPointer` and `masterPromptFileName` go from `integration.go`,
and `Run` puts the rendered Master prompt straight into `spec.Prompt` without writing `master.md`.

Every role's first user message changes from the prompt to the pointer;
the decision record lists this as an open risk.

Tests:
`settings_test.go`'s `TestPrepare_PromptLaunchLimit` becomes a prompt over 30000 bytes that launches, with the launch line carrying the pointer to `prompt.md` and not the prompt text, and `prompt.md` holding the full prompt.
`command_test.go` pins the pointer argument on the launch line for posix and pwsh.
`runlevel_test.go` pins that Master's `Spec.Prompt` is the rendered prompt and that no `master.md` is written.

**ImpactSummary:** every shuttle-launched agent (all loom, batten, burler, webster and orch roles) receives the pointer as its first message; webster loses its private pointer file.
