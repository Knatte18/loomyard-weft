# A landing notice on request, per task

`landing-watch` · Task · feature, dobby, creel, orch, board, reed, config · running · Drift-Baseline

Depends on: [`adopt-check-visible`](design-adopt-check-visible.md)

## Seen (operator, 2026-10-28)

A run that lands sends nothing: the orch learns of a landing only when something else wakes it, and the orch never polls runs.
When the operator asks the orch to report a run's result as soon as it lands, the orch cannot keep that promise, and the operator has to ask again.

## Design (operator, 2026-10-28 to 2026-10-31)

Dobby sends the notice, never batten: dobby drives each run to its end, knows its outcome and landing commit, and already mails the orch.
No process of its own is needed, since dobby only reads the run state it already has.

One setting in the dobby's own config decides which runs get a notice: the `beam` module (`beam.yaml`, beside `max_runs`) in the worktree's `_lym/config`, so each beam's dobby has its own.
It takes three values:
- `all`: a notice for every run that dobby drives;
- `per-task`, the default: a notice only for a task whose board entry carries a field such as `notify_on_land`, set with `lym board upsert`; the field is cleared once its notice has gone out;
- `off`: no notice at all, a task's field included.

The notice:
- When the run lands, or ends without landing, dobby posts one short creel line to the orch naming the slug, the outcome and the landing commit.
- A beam counts as a run: a notice also goes out when a dobby lands a beam into its parent after `lym beam ready`.
- The test for a run past its Land row is the one `lym hibernate request --after` already has (`landFinished` in `internal/hibernateengine`); it is taken out into one shared piece that hibernate and dobby both use, never copied, and it also tells a run that landed from one that ended without landing, since the notice names the outcome.

The worktree's reed overview, which already shows its runs and its dobby, shows which of the three values the dobby's setting holds, and which tasks carry the field.
