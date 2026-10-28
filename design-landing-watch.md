# A landing notice on request, per task

`landing-watch` · Note · feature, workspaces, creel, orch

## Seen (operator, 2026-10-28)

A run that lands sends nothing: the orch learns of a landing only when something else wakes it, and the orch never polls runs.
When the operator asks the orch to report a run's result as soon as it lands, the orch cannot keep that promise, and the operator has to ask again.

## Idea

The orch can ask for a landing notice on one slug: a watch set per task, never on by default for every run.
- The orch sets the watch when the operator asks, through a verb of its own, and it can be removed again.
- Dobby sends the line, not batten (operator, 2026-10-28): dobby drives each run to its end and already mails the orch, so batten knows nothing of who listens.
- When the watched run lands, or ends without landing, one short line reaches the orch through creel, naming the slug, the outcome and the landing commit.
- The watch ends when it has fired once.

## Open

- Where dobby keeps the watch: a flag on the board entry or a record of its own.
