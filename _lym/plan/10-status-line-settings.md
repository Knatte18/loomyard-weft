# Card 10 — Sessions on a listed account get the lym status line

**Create:**
- `plan:internal/shuttleengine/claudeengine#statusLineCommand` -> `func statusLineCommand(lymPath, account string) string`
- `plan:internal/shuttleengine/claudeengine#addStatusLine` -> `func addStatusLine(settings []byte, command string) ([]byte, error)`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#Claude.OpenLaunch`
- `internal/shuttleengine/claudeengine#writeOpenFiles` -> `func writeOpenFiles(dir, creelFlagDirPosix string, scope sessionScope, statusLine string) (settingsPath, appendPromptPath string, err error)`
- `internal/shuttleengine/claudeengine#TestOpenLaunch`
- `plan:internal/shuttleengine/claudeengine#TestPrepare_ExportsTheResolvedAccount`

**Extends:**
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `plan:internal/shuttleengine/claudeengine#Claude.resolveLaunchAccount`
- `internal/shuttleengine/claudeengine#shQuote`
- `internal/shuttleengine/claudeengine#LymBinary`
- `internal/shuttleengine/claudeengine#TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`

**Intent:**
D5's status line.
`statusLineCommand` is `'<lym>' claude statusline '<account>'`, the lym path from `LymBinary` (`selfexec.LymBinary`) and both values `shQuote`d; Claude Code runs it in the session's directory, the worktree's.
`addStatusLine` sets the settings document's `statusLine` key to `{"type": "command", "command": <command>}` and leaves every other key as it is.
`Prepare` adds it to `settings.json` when card 6's launched account is a listed one, and adds nothing for an unlisted directory.
`OpenLaunch` adds it to its settings whenever its launched account is listed: `writeOpenFiles` takes the status line.
A listed account needs the machine layer, which loads only for an engine told a board directory, the same condition that gives it a creel flag directory; so an engine told none lists no account and writes nothing, as today, and `buildOpenSettings` is unchanged.
The lym status line ranks above the operator's own and replaces it in lym panes, unchained (Auto-mode assumption).
`PATTERN-engine-parity`: the status line is an engine settings item and is not compared, so Codex owes no counterpart.

`TestPrepare_ExportsTheResolvedAccount` (card 6) gains rows: a session on a listed account gets `statusLine.command` naming the lym path, `claude statusline` and its account; a session on an unlisted directory gets no `statusLine` key; every hook the settings held before is still there.
`TestOpenLaunch` gains rows: a listed account's settings hold the creel hooks and the status line; an unlisted directory's hold the creel hooks alone.

**ImpactSummary:** One settings key added after the settings are built; `writeOpenFiles`' one caller is `OpenLaunch`; the creel hook layout is unchanged.

**Commit:** 10: claudeengine: the lym status line for sessions on a listed account
