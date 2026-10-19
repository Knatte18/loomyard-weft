# orch status reports a live strand after the orch Claude session exits, so --continue is refused

`orch-strand-live-after-exit` · Note · bug, orch, reed

Imported from [issue #746](https://github.com/Knatte18/loomyard/issues/746).

Seen on Linux (tmux 3.4) in the loomyard hub's prime.
The operator closed the orch Claude session by mistake and then talked to the same session id outside reed.

`lyx orch status` afterwards reported `strand_live: true` with `watcher_live: false`, while the orch strand's pane (`%3`) held only a `bash` shell (`tmux list-panes` showed `%3 0:bash bash`, so the pane also sat under a session and window named `0` and `bash`, not the prime's `loomyard:orch`).
`lyx orch start --help` says `--continue` and `--adopt` are refused while the strand is live, so getting back needed `lyx orch stop` first.

Workaround: `lyx orch stop`, then `lyx orch start --continue`, then `lyx reed attach`.

Expected:
- A strand whose pane no longer runs its Claude session counts as dead, so `lyx orch status` shows it and `lyx orch start --continue` resumes it in one step.
- The watcher's death is surfaced or the watcher is restarted, instead of the orch going silent.
- The orch pane stays in the prime's reed session and window.