# Card 11 — claude-name-flag

**Edit:**
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine/command_test.go`

**Uses:**
- `internal/agentname/agentname.go`
- `internal/shell/shell.go`

**Intent:**
Name the Claude session after its strand.
`buildLaunchCmd` and `buildResumeCmd` each append `--name ` followed by `sh.EnvRef(agentname.StrandNameEnv)`.
It must be a reference, not a value: shuttle builds both lines before reed forms the name, and the pane shell expands it from the export reed's launch script makes.
The resume line carries it too, because reed replays that line verbatim on every resume and Claude would otherwise come back unnamed.
Both builders' doc comments say why the flag is a reference.

`command_test.go`: for the POSIX and pwsh dialects, the launch and the resume line each contain exactly `--name "${LYX_STRAND_NAME}"` and `--name "$env:LYX_STRAND_NAME"` respectively, in the position the test pins for the other flags.

**ImpactSummary:** Adds one flag to the two Claude command-line builders; every Claude strand launch and resume carries it.

**Commit:** 11: claude engine: name the session after its strand
