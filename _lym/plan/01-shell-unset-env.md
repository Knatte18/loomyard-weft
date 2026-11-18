# Card 1 — Shell statement that unsets an environment variable

**Extends:**
- `internal/shell#Shell`

**Create:**
- `internal/shell#posixShell.UnsetEnv`
- `internal/shell#pwshShell.UnsetEnv`
- `internal/shell#TestShell_UnsetEnv`

**Uses:**
- `internal/shell#posixShell.ExportEnv`
- `internal/shell#pwshShell.ExportEnv`

**Intent:**
Add `UnsetEnv(key string) string` to the `Shell` interface: a standalone statement that removes key from the shell session, the counterpart of `ExportEnv`.
The POSIX dialect returns `unset KEY`; the pwsh dialect returns `Remove-Item -ErrorAction SilentlyContinue Env:KEY`, so an absent variable is no error.
The key is an environment name the caller owns and is written as `ExportEnv` writes it.
Card 6 chains it ahead of a Claude launch line whose account is on `~/.claude` (D4), so the line runs with `CLAUDE_CONFIG_DIR` unset whatever the `env` module exported before it.
`TestShell_UnsetEnv` is a tier0 table over both dialects, as `TestShell_ExportEnv` is: it pins each dialect's statement, and that `Chain(UnsetEnv(k), cmd)` keeps the statement ahead of the command.

**ImpactSummary:** One interface method added; `posixShell` and `pwshShell` are its only implementers, so no other package changes.

**Commit:** 1: shell: add UnsetEnv, the statement that removes a variable from the session
