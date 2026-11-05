# Codex sessions get creel mail through hooks, as Claude sessions do

Creel's hook delivery reached only Claude sessions.
Every Codex strand was a pane-mail strand, and mail from another agent arrived as a `run lym creel read` reminder.
`codexengine` now carries creel's delivery the way `claudeengine` does, and a Codex launch built for a hub is marked `CreelHooks`.

## What changes

- **Tool boundary.**
  A second `PostToolUse` entry, after the activity hook, forms the strand's flag name and runs `lym creel hook-read codex` only when that flag stands in the hub's creel flag directory.
  With no flag it prints the empty hook output and starts no process.
- **Turn end.**
  The `Stop` recording is one merged command.
  When the flag stands it pipes the payload to `hook-read` and exits 0 if that handed out mail, recording no turn end, since the block continues the turn; otherwise it appends the `Stop` record as before.
- **Hook output.**
  `Codex.HookOutput` hands mail over as `{"decision":"block","reason":<mail>}`.
  It drops the echo, because the idle wake already shows the same lines to the operator as a user turn.
- **Idle wake.**
  The `Stop` setting also carries an async `lym creel hook-wait codex` waiter, with a timeout of a day.
  No exit of a Codex hook wakes an idle session, so on a ring `Codex.WakeIdle` runs `codex queue --thread <session_id> --message <mail>` before creel marks the mail read.
  A failed queue leaves the mail unread, and creel's bell into the pane wakes the session instead.
- **Trust sentence.**
  The prompt file now opens with the creel trust sentence after the deny notice.
  Without it Codex reads a hook's block as a failed tool call.
- The package doc's "no creel hooks" goes, and the README and overview now say Codex's hooks carry creel mail, under trial.

Shared pieces moved from `claudeengine` into `shuttleengine` (`creelhooks.go`): `CreelTrustSentence`, `CreelFlagDir` and `StrandFlagNameShell`.
`shuttleengine` gains the optional `IdleWaker` interface beside `ToolBoundaryContextRenderer`.
`creelcli`'s `hook-wait` calls the engine's `WakeIdle` when the engine implements it and exits 0, and otherwise writes the wake text to stderr and exits with the wake code as before.

## Design decisions

- **The idle wake runs in the strand's hook-wait waiter, not in the creel daemon.**
  Claude's idle wake already rides that waiter, so this keeps engine-specific commands out of the daemon.
  The Goal's wording, "`codex queue` from the daemon", is amended to match; the effect, an idle Codex session woken by mail, is the same.
- The waiter's command sets `CODEX_HOME` and `LYM_CODEX_BINARY` itself, so the queued turn reaches this session's home and the binary `codex.yaml` names, even when Codex hands its hooks a trimmed environment.
- An engine told no board directory (a standalone run) and `OpenLaunch` carry no creel hooks.
  An opened session has no prompt file to hold the trust sentence, so it stays a pane-mail strand.
- A block after a tool call makes Codex read that call as failed, so the session may run the call once more.

## To check

- `Codex.WakeIdle` and the waiter live in the engine and `hook-wait`, not in the daemon: confirm this is the intended placement (`internal/shuttleengine/codexengine/capabilities.go`, `internal/creelcli/hookwait.go`).
- Unit tests cover the hook output, the launch settings and the wake call (`hookrun_test.go`, `capabilities_test.go`, `hookread_test.go`).
  No live check on real Codex sessions was run; that is the separate trial task that follows.
