# Card 1 — claudeengine-deny-notice

**Edit:**
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine/settings_test.go`

**Create:**
- `internal/shuttleengine/claudeengine#buildDenyNotice`

**Uses:**
- `internal/shuttleengine/claudeengine#buildSettings`

**Intent:**
Add the system-prompt notice that announces each standing tool deny the engine installs (decision record D6, D7).
The notice is built here, beside the deny hooks, so the two cannot drift.

In `settings.go`, add three notice constants beside the existing steer constants, one per announced deny:

- Agent deny, non-fork run: the `Agent` tool is unavailable in this session; do all exploration and work in this session, with no subagents.
- Agent deny, fork run: the `Agent` tool accepts only fork subagents and refuses every other subagent type; a fork does its own work and never spawns further subagents.
- AskUserQuestion deny: `AskUserQuestion` is unavailable; when blocked or needing operator input, state the question as the final message and end the turn without writing the result file.

Each sentence must hold in every session that receives it, forks included, since forks inherit the system prompt.
The webster fork-context guard is never announced.

Add `buildDenyNotice`, which returns the sentences for the denies `buildSettings` installs under the same `(interactive, cfg, forkSubagents)` inputs, joined by a single space, or the empty string when none is installed:
the Agent sentence (fork or non-fork form) exactly when `cfg.ClaudeDenyAgentTool` is set, and the AskUserQuestion sentence exactly when the run is autonomous and `cfg.ClaudeDenyAskUserQuestion` is set (an interactive run's AskUserQuestion marker hook records, never denies, so it is not announced).
Keep the two install conditions in one place both functions read, rather than restating them, so a change to one cannot leave the other behind:
move them out of `buildSettings`' inline `if` tests into one unexported helper over `(interactive, cfg)`, and reshape `buildSettings`' body to read that helper, leaving its output byte-identical.

Extend the package `init` forbidden-character check to the three notice constants, and make it also reject a line break (`\n`, `\r`), since the assembled notice rides one command-line argument that reed types with `tmux send-keys`.
Update the file comment to mention the notice.

Tests in `settings_test.go`:

- For every combination of `ClaudeDenyAgentTool`, `ClaudeDenyAskUserQuestion`, `interactive` and `forkSubagents`, parse `buildSettings`' output and assert the notice carries the Agent sentence if and only if a `PreToolUse` entry with matcher `Agent` is installed, and the AskUserQuestion sentence if and only if the AskUserQuestion entry installed is the deny (not the interactive marker); the notice is empty exactly when neither is installed.
- The fork-run notice names fork subagents as the permitted type and tells a fork not to spawn further subagents; the non-fork notice names the `Agent` tool as unavailable; the AskUserQuestion notice states the final-message channel.
- No notice mentions `lyx webster`.
- Every assembled notice is free of line breaks, single quotes, double quotes and backslashes.

**ImpactSummary:** new unexported constants and functions in `claudeengine`, plus a refactor of `buildSettings`' body onto the shared install-condition helper; its output stays byte-identical, pinned by the existing `settings_test.go` cases, and no caller outside the package exists.
