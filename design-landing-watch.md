# A landing notice on request, per task

`landing-watch` · Note · enhancement, batten, creel, orch

## Seen (operator, 2026-10-28)

A run that lands sends nothing: the orch learns of a landing only when something else wakes it, and the orch never polls runs.
When the operator asks the orch to report a run's result as soon as it lands, the orch cannot keep that promise, and the operator has to ask again.

## Idea

The orch can ask for a landing notice on one slug: a watch set per task, never on by default for every run.
- The orch sets the watch when the operator asks, for example `lym batten watch <slug>`, and it can be removed again.
- When the watched run lands, or ends without landing, one short line reaches the orch through creel, naming the slug, the outcome and the landing commit.
- The watch ends when it has fired once.

## Open

- Where the watch lives: a flag on the board entry, a record batten's Land row reads, or a creel subscription per slug.
- Whether the orch's watcher or batten's Land row sends the line.
