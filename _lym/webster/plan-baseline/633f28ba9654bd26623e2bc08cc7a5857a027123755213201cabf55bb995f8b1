# Card 6 — Launches resolve the account, export its directory and record it

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#Claude.OpenLaunch`
- `internal/shuttleengine/claudeengine#buildLaunchCmd` -> `func buildLaunchCmd(sh shell.Shell, bin, pointer, settingsPath, sessionID, model, effort, appendPromptPath, envFilePath, promptCacheTTL, remoteControlName string, account accountEnv, resume, skipPermissions, forkSubagents bool) string`
- `internal/shuttleengine/claudeengine#buildResumeCmd` -> `func buildResumeCmd(sh shell.Shell, bin, settingsPath, sessionID, model, effort, appendPromptPath, envFilePath, promptCacheTTL, remoteControlName string, account accountEnv, skipPermissions, forkSubagents bool) string`
- `internal/shuttleengine/claudeengine#ScrubbedEnv`
- `internal/shuttleengine/claudeengine#TestBuildLaunchCmd`
- `internal/shuttleengine/claudeengine#TestBuildResumeCmd`
- `internal/shuttleengine/claudeengine#TestBuildResumeCmd_NoticePosix`
- `internal/shuttleengine/claudeengine#TestOpenLaunch`
- `internal/shuttleengines#TestEnvScrub_MatchesTmuxkitServerEnviron`
- `internal/testkit/tmuxkit#ServerEnviron`

**Create:**
- `plan:internal/shuttleengine/claudeengine#accountEnv` -> `type accountEnv struct`
- `plan:internal/shuttleengine/claudeengine#accountEnv.wrap` -> `func (e accountEnv) wrap(sh shell.Shell, cmd string) string`
- `plan:internal/shuttleengine/claudeengine#launchedAccount` -> `type launchedAccount struct`
- `plan:internal/shuttleengine/claudeengine#launchedAccountFileName` -> `const launchedAccountFileName = "account.json"`
- `plan:internal/shuttleengine/claudeengine#writeLaunchedAccount` -> `func writeLaunchedAccount(dir string, launched launchedAccount) error`
- `plan:internal/shuttleengine/claudeengine#readLaunchedAccount` -> `func readLaunchedAccount(dir string) (launchedAccount, bool, error)`
- `plan:internal/shuttleengine/claudeengine#Claude.resolveLaunchAccount` -> `func (c *Claude) resolveLaunchAccount(claude Config) (accountEnv, launchedAccount, error)`
- `plan:internal/shuttleengine/claudeengine#TestPrepare_ExportsTheResolvedAccount` -> `func TestPrepare_ExportsTheResolvedAccount(t *testing.T)`

**Extends:**
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shell#Shell`
- `internal/envconfig#LoadConfig`
- `plan:internal/shuttleengine/claudeengine#resolveAccount`
- `plan:internal/shuttleengine/claudeengine#readAccountFile`
- `plan:internal/shuttleengine/claudeengine#readUsage`
- `plan:internal/shuttleengine/claudeengine#canonicalConfigDir`
- `internal/shuttleengine/claudeengine#claudeHomeDir`
- `internal/shuttleengine/claudeengine#TestPrepare_ThreadsSpecIntoLaunchCmds`

**Intent:**
D4 at every launch the claude engine builds: a fresh one, a resume, and `OpenLaunch`'s lines.

`Claude.resolveLaunchAccount` reads the account file and usage under the told `Worktree.AccountDir` and calls `resolveAccount` (card 4).
The returned `accountEnv` says one of: export a directory (an account whose canonical directory is not `~/.claude`'s), unset (an account on `~/.claude`, since setting it there moves `.claude.json`), or nothing (no account).
`accountEnv.wrap` applies it to a line: an export wraps the finished line with `sh.WithEnv("CLAUDE_CONFIG_DIR", dir, ...)` outermost, and an unset chains `sh.UnsetEnv("CLAUDE_CONFIG_DIR")` (card 1) ahead of it.
Both sit inside reed's launch script after the `env` module's exports (`hubgeom` `paneEnv` through reed's `paneEnvExports`), so they win over an `env` value; with nothing, the `env` value or the default stands.
The returned `launchedAccount` holds `ConfigDir`, the launch's effective directory (the exported one, else the `env` module's `CLAUDE_CONFIG_DIR` from `envconfig.LoadConfig(c.dirs)`, else `~/.claude`), and `Account`, the listed account whose directory equals it compared by `canonicalConfigDir`, empty when none does.
A config load failure refuses the launch: a bad value never degrades to no accounts (D1); an undecodable account or usage file reads as empty (card 4).

`Prepare` resolves before it writes any artifact, passes the `accountEnv` to `buildLaunchCmd` and `buildResumeCmd`, and writes `account.json` into the run directory through `internal/state` (`PATTERN-tolerant-records`).
`OpenLaunch` chains the statement form, `sh.ExportEnv` or `sh.UnsetEnv`, after the TTL export ahead of both its `Cmd` and its `ResumeCmd`, so the resume line's fresh fallback runs under it too, and, for an engine told a hub board directory, writes `account.json` into its session directory, creating it.
Cards 7, 10, 13 and 14 read that record as the session's account wherever D5, D7 and D8 need one, also for a worktree with no entry that runs on a listed directory.
Reed's relaunch replays the stored line, so a strand keeps the account it was launched under until an engine-built launch starts it again.

`ScrubbedEnv` gains the name `CLAUDE_CONFIG_DIR`, so a tmux server spawned from a pane on one account never carries it into a pane with no export; `tmuxkit.ServerEnviron`, the test kit's mirror of the server environment, strips it too.

`TestPrepare_ExportsTheResolvedAccount` is tier0 over a temporary hub, `HOME` and fake config directories, built like `TestPrepare_ThreadsSpecIntoLaunchCmds`: an entry on a non-default directory puts `CLAUDE_CONFIG_DIR=<dir>` on the launch and resume lines; an account on `~/.claude` unsets it; no account leaves both lines without either; a hub pin beats an entry and a beams pin; a beams pin applies to a prime and a beam and not a task; a beams pin with a future `blocked_until` yields to the entry and stands with no entry; a hub pin with one stands; an entry naming a removed or retired account launches with neither export nor unset; an engine told no worktree resolves only the hub pin; `account.json` names the effective directory and the listed account, also for an entry-less worktree whose `env` module names a listed directory; an invalid machine-layer list refuses `Prepare` before any artifact; with one account of positive weight a hub pin naming it is still exported; an orch-role spec and a dobby-role spec told the prime's worktree under a beams pin export the pinned account (D9); with the `env` module's `CLAUDE_CONFIG_DIR` naming another directory, the line still carries the account's export and `account.json` names the exported directory.
`TestBuildLaunchCmd`, `TestBuildResumeCmd` and `TestBuildResumeCmd_NoticePosix` pass the zero `accountEnv` and gain an export row and an unset row.
`TestOpenLaunch` gains the export, unset and none rows on both lines, the fresh fallback of `ResumeCmd` included, and the `account.json` row.
`TestEnvScrub_MatchesTmuxkitServerEnviron` adds `CLAUDE_CONFIG_DIR=/x` to its environ, which both sides must strip, and asserts `EnvScrub().Apply`'s output holds no `CLAUDE_CONFIG_DIR`.

**ImpactSummary:** The two command builders gain a parameter whose callers are `Prepare` and three tests in the package; every launch gains at most one statement; the reed server scrub and its test mirror gain one name.

**Commit:** 6: claudeengine: export the resolved account's CLAUDE_CONFIG_DIR and record it per run
