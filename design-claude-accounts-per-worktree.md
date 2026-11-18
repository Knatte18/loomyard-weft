# Claude accounts per worktree: pick, pins, headroom, transcript carry and switch on a usage limit

`claude-accounts-per-worktree` · Task · feature, claude, shuttle, reed, orch, batten, cwd, config, tools

Decision record: [Sketch](decision-claude-accounts-per-worktree.md)

Depends on: [`leftover-strands-and-runs`](design-leftover-strands-and-runs.md), [`records-push-main-weft-ended`](design-records-push-main-weft-ended.md)

Filed from the Sketch `claude-account-pick-simple`.

Takes: the Sketch's decision record; main with leftover-strands-and-runs and shuttle-start-idle-and-prompt landed, which change shuttle's run tracking, startup wait and prompt delivery in internal/shuttleengine that D7's relaunch builds on
Delivers: machine-local Claude accounts and pins, a per-hub account file with the weighted pick at worktree making, account resolution and export at every claude engine launch, headroom from the status line and limit hits, transcript carry on resume, readers over every known config directory, and the switch on a usage limit with shuttle's in-run relaunch and the orch watcher's restart
Exports: claude module keys accounts and account_pins (machine layer); lym claude account set and show; the status line verb; reed ops that end a strand's process while holding it and relaunch its pane under the same GUID; shuttle's stop verb relaunch mode and one running-run predicate on the run record

The part changes these packages:

- internal/shuttleengine/claudeengine (about 900 changed lines)
- internal/shuttleengine (about 650 changed lines)
- internal/reedengine (about 200 changed lines)
- internal/claudecli (about 200 changed lines)
- internal/orchcli (about 150 changed lines)
- internal/orchengine (about 80 changed lines)
- internal/shuttleengines (about 40 changed lines)
- internal/battencli (about 40 changed lines)
- internal/adoptcli (about 30 changed lines)
- internal/configengine (about 30 changed lines)
- tools/tokencount (about 20 changed lines)
- internal/lymdirs (about 10 changed lines)
- cmd/lym (about 5 changed lines)

## Decision (operator, 2026-11-18): the pick takes the account with the most quota left



This replaces the weighted counter of D3 in the stored decision record.



- Among the eligible accounts (listed, positive weight, logged in, with headroom in every window they list), take the one whose tightest window has the most left.

- The weighted counter stays only as the tie-break, and for accounts with no headroom reading yet.

- Pins are unchanged: a hub pin, then a beams pin, win over the pick.

  That covers a repository always on account A, one always on account B, and alternating repositories whose prime and beams stay on A.



Discussion rewrites D3, and what reads it in D2 and D5, to fit.
