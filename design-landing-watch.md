# A landing notice on request, per task

`landing-watch` · Note · feature, dobby, creel, orch

## Seen (operator, 2026-10-28)

A run that lands sends nothing: the orch learns of a landing only when something else wakes it, and the orch never polls runs.
When the operator asks the orch to report a run's result as soon as it lands, the orch cannot keep that promise, and the operator has to ask again.

## Design (operator, 2026-10-30)

The wait already exists in `lym hibernate request`: its detached daemon waits until a named batten run has finished its Land row (`--after <slug>`), or until no batten run is in flight (`--after-all`), and then mails the orch through creel.
The landing notice reuses that wait: it is taken out of the hibernate package into one shared piece that both the hibernate daemon and a new watch use, never copied.

- The orch sets a watch when the operator asks, through a verb of its own, for one slug or for no run in flight; it is never on by default for every run, and it can be withdrawn.
- A watch is a detached process like the hibernate daemon, which waits on the shared trigger, posts one short creel line to the orch naming the slug, the outcome and the landing commit, and ends.
- The watch ends when it has fired once.
- Hibernate keeps its own behaviour: after the trigger it pauses the hub and hibernates; only the wait is shared.

## Open

- Whether the shared trigger already tells a run that ended without landing from one that landed, or must learn to, since the notice names the outcome.
- The verb's name and whether it lives beside `lym hibernate` or in `lym beam`.
