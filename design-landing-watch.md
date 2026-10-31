# A landing notice on request, per task

`landing-watch` · Task · feature, dobby, creel, orch, board, reed, config · draft

## Seen (operator, 2026-10-28)

A run that lands sends nothing: the orch learns of a landing only when something else wakes it, and the orch never polls runs.
When the operator asks the orch to report a run's result as soon as it lands, the orch cannot keep that promise, and the operator has to ask again.

## Design (operator, 2026-10-28 to 2026-10-31)

Dobby sends the notice, never batten: dobby drives each run to its end, knows its outcome and landing commit, and already mails the orch.
No process of its own is needed, since dobby only reads the run state it already has.

A notice goes out in two cases, both supported:
- Per task: a field on the task's board entry, such as `notify_on_land`, set with `lym board upsert`, asks for a notice when that run ends; it ends when it has fired once.
- Per dobby: a switch in the dobby's own config, the `beam` module (`beam.yaml`, beside `max_runs`) in the worktree's `_lym/config`, asks for a notice on every run that dobby drives; it is off by default, and each beam's dobby has its own.

The notice:
- When the run lands, or ends without landing, dobby posts one short creel line to the orch naming the slug, the outcome and the landing commit.
- A beam counts as a run: a watch or the switch also fires when a dobby lands a beam into its parent after `lym beam ready`.
- The test for a run past its Land row is the one `lym hibernate request --after` already has (`landFinished` in `internal/hibernateengine`); it is taken out into one shared piece that hibernate and dobby both use, never copied, and it also tells a run that landed from one that ended without landing, since the notice names the outcome.

The worktree's reed overview, which already shows its runs and its dobby, shows whether the dobby's switch is on, and which tasks carry a per-task watch.
