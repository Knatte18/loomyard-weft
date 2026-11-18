# Claude accounts per worktree: pick, pins, transcript carry, limit switch

Each worktree now runs its Claude sessions under one of the Claude accounts logged in on the machine.
The account is picked by weight and headroom when the worktree is made, can be pinned per hub or for the prime and beams, and moves to another account when its own hits a usage limit.
This spreads Claude usage over the accounts instead of draining `~/.claude` alone.
The whole policy is machine-local; no tracked config or git file holds any of it.

Closes: no issue.

## What changes

- `claude` module: new machine-layer keys `accounts` (name, `config_dir`, `weight`, `windows`) and `account_pins.hub` / `account_pins.beams`.
  A non-empty value in a tracked `claude.yaml` is refused, as is an invalid list or a pin naming no listed or a retired account.
  Weight 0 retires an account: never picked, but its directory is still read.
- A per-hub cache, `<Bolt>/.lym/claude/accounts.json`, maps worktree name to account name and holds the pick counter.
  Usage records sit beside it in `usage/<account>.json`.
  A missing or wiped file leaves every worktree on today's behaviour.
- The pick runs at the task run's Worktree-Create step, at a new beam's creation and at each worktree `lym adopt` adds.
  It takes the lowest count over weight among logged-in accounts with headroom, and falls back to the earliest blocking reset when none has headroom.
  A pin skips it, and the prime is never picked.
- Every claude engine launch resolves the account (hub pin, beams pin, file entry) and exports `CLAUDE_CONFIG_DIR`, or unsets it for an account on `~/.claude`.
  The run directory records the launched account.
  `CLAUDE_CONFIG_DIR` joins the engine's scrub list, so reed's tmux server no longer inherits it from a shell.
- Headroom comes from a status line command (a new `lym claude` verb) that merges `rate_limits` figures into the account's usage file, and from `blocked_until` recorded on a usage-limit hit.
- A resume first copies the newest copy of its transcript, with its session directory, into the directory of the account it runs under.
  A copied resume whose first turn ends on a non-limit API error starts the session fresh.
- A usage limit switches the worktree to another eligible account.
  Shuttle relaunches the strand inside the same run as a resume of the same session id; the orch watcher restarts the orch the same way.
  A hub pin never switches; a beams pin switches while its account is blocked.
- `lym claude account set <account>` and `lym claude account show` set and display a worktree's account.
- Readers of Claude's config directory (`LocateSessions`, the fork audit, liveness probing, `tools/tokencount`) read every known directory.
- `docs/reference/claude-accounts.md` documents the keys, the files and the verbs.

## Design decisions

- Shuttle has no in-run relaunch before this change.
  The stop verb gains a relaunch mode that marks the run record relaunching and ends the process while reed holds the strand under its GUID.
  Reed gains two ops for this: end a strand's process while holding it, and relaunch its pane under the same GUID with a new command and session id.
- One predicate on the run record reads a relaunching run as live within a bound of twice the startup timeout.
  Past the bound an attach or `Wait` ends the run as a strand that died, or attaches when the strand is live.
  `PATTERN-shuttle-provider-seam` is amended to admit the relaunch mode.
- The relaunch's startup wait decides on the pane capture alone, because the kept events file already holds the limited session's start lines.
- Copying a transcript across accounts drops thinking blocks the second organization cannot honour; messages are kept.
  Both accounts need the same plugins, `enabledPlugins` and MCP servers; the reference doc gives the wiring.
- The lym status line replaces the operator's own in lym panes, and any shell that can run the status line verb can forge headroom figures.
  Forged input never writes or clears a `blocked_until`.

## Check by hand

- The board entry records a later operator decision that the pick take the account with the most quota left, the counter staying as tie-break.
  The diff implements the weighted counter (`internal/shuttleengine/claudeengine/accountpick.go`); confirm whether that decision lands here or in a follow-up.
- Manual check on a machine with two accounts, not run: log the second account in with `CLAUDE_CONFIG_DIR=<dir> claude` and `/login`,
  link its `plugins` and `settings.json` to `~/.claude`, copy the MCP servers into its `.claude.json`, and list both with `lym config claude --layer machine`.
  Start two task runs, confirm each worktree's entry and that a transcript lands in each account's `projects/`,
  then run `lym claude account set` in one worktree and resume a session there.
  This is the first real resume of a copied transcript under another account.
- `missing-sessions-dir-reads-unreadable`: a registry directory that does not exist reads as unreadable in the liveness read, which can leave a dead session unproven.
- `record-before-relaunch-order-untested`: no test pins that the run record names the new launch before reed relaunches the pane.
- `stop-ended-start-path-relaunch-outcome-not-a-result`: a stop that ends a start-path relaunch returns `OutcomeDied` and, on the start path, an error wrapping `ErrNotStarted`; check this return shape in `internal/shuttleengine/accountrelaunch.go`.
- Headroom is per hub while accounts are shared across hubs, so a hub can pick an account another hub has spent.
- If Claude Code renames the status line fields, the verb writes nothing and the pick falls back to weights and limit hits.
