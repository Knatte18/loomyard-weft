# Two Claude accounts, each worktree on one of them by a configured weight

`claude-account-weights` · Note · feature, claude, shuttle, config

## Idea (operator, 2026-11-02)

Spread Claude usage over two accounts, so one account's limits are not the bottleneck: each worktree uses one of the accounts, chosen by a weight.
The weights are config, for example account A twice for each use of account B, so account A carries twice the load of B on average.

## Accounts of different plans (operator, 2026-11-03)

The accounts need not be of one plan: for example a Claude Ultra account beside a Max account with about half its usage.
The plans' limit windows differ too: the operator reports that Max has a five-hour window that Ultra lacks.
So a fixed weight alone is not enough: a Max account can hit its five-hour window while its weekly share is far from spent.

- The weight sets the long-run share, from each account's plan.
- The pick also reads each account's current headroom in every window it has, and passes over an account that is near or at a limit; an account with no five-hour window is never judged by one.
- Lym today learns of a limit only when a pane shows the usage-limit message (`internal/shuttleengine/claudeengine/usagelimit.go`); reading an account's headroom before a pick is new, and the source is open: Claude Code's status line input, `/usage`, or lym's own transcript totals per config directory against a configured budget. Check what Claude Code exposes before choosing.
- This was weighed against offloading to Codex (`codex-engine`): the two-account route keeps every agent on Claude's review quality and is far smaller to build, so it is the first route to quota relief (orch, 2026-11-03).

## Seen (orch, 2026-11-02)

Lym cannot pick an account per worktree today.
`lym config claude` has no key for an account or a config directory, and a search of `internal/` found no use of `CLAUDE_CONFIG_DIR`, so every session runs under whichever account the machine is logged in to.

Claude Code is believed to keep an account's login in a config directory that the `CLAUDE_CONFIG_DIR` environment variable selects.
That is not verified: check it against the Claude Code documentation before anything is built.

## Built since (landed 2026-11-03)

`lym config env` (`0d155d1a`) holds per-machine environment variables that lym exports to every process it starts for the hub: gate tiers, Drift steps and agent session panes.
A pane receives them through the `PaneEnv` closure on reed's geometry, exported ahead of the strand's launch command and read afresh at each launch.
It refuses `PATH` and any `LYM_` name, and `CLAUDE_CONFIG_DIR` is allowed.

So a machine-wide `CLAUDE_CONFIG_DIR` can be set today, which moves every session to one other account.
It is one value for the whole hub: the per-worktree pick, the weights and the headroom check below are still unbuilt, and a per-worktree value needs a source beside this module's fixed map.

## Design questions for a Discussion

- Config: a list of accounts, each with a name, a config directory, a weight and its limit windows, in the claude config; a machine-layer key, since the accounts belong to the machine.
- Choice: per worktree, never per session, so all of a run's sessions (writer, reviewers, fixers, driver) share one account.
  The pick is made once, when the worktree's first session starts, and recorded with the worktree so a resume, a re-attach and a hand pass keep it.
  With headroom in the pick, a deterministic hash of the slug no longer suffices: a persisted weighted counter that skips an account short of headroom is the candidate.
- Scope: the prime's orch and each beam's orch and dobby need a rule of their own, since they outlive a task run.
- A session spawn sets the chosen account's environment in one place in the claude engine's launch, so no caller sets it.
  The pane launch now already exports the `env` module's variables ahead of the launch command (`reedengine/panebin.go`); the account's variable would sit beside them, and its precedence over the module's is to be decided.
- A usage limit on one account: whether lym moves a worktree to the other account when one is limited, or only reports it; the first needs the session's history to be readable under the other account, which it is not by default.
- Cost and quota reports (`lym report`, `tokencount`) read transcripts per config directory, and would need to read both.

## Not in scope

Logging the accounts in, and whether the use is within the accounts' and the organisation's terms, which is the operator's to settle.
