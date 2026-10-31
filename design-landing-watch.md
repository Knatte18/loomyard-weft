# A landing notice on request, per task

`landing-watch` · Task · feature, dobby, creel, orch · draft

## Seen (operator, 2026-10-28)

A run that lands sends nothing: the orch learns of a landing only when something else wakes it, and the orch never polls runs.
When the operator asks the orch to report a run's result as soon as it lands, the orch cannot keep that promise, and the operator has to ask again.

## Design (operator, 2026-10-28 and 2026-10-30)

Dobby sends the notice, never batten: dobby drives each run to its end, knows its outcome and landing commit, and already mails the orch.
No process of its own is needed, since dobby only keeps a list of watches and reads the run state it already has.

- The orch sets a watch when the operator asks, through a verb of its own, for one slug; it is never on by default for every run, and it can be withdrawn.
- When the watched run lands, or ends without landing, dobby posts one short creel line to the orch naming the slug, the outcome and the landing commit.
- The watch ends when it has fired once.
- The test for a run past its Land row is the one `lym hibernate request --after` already has (`landFinished` in `internal/hibernateengine`); it is taken out into one shared piece that hibernate and dobby both use, never copied.
- The hibernate daemon stays as it is: only its wait is shared.

## Open

- Where dobby keeps the watch: a flag on the board entry or a record of its own.
- The shared test says only that a run is past Land, so it must also tell a run that landed from one that ended without landing, since the notice names the outcome.
- The verb's name and where it lives.
- Whether the hibernate request should move into dobby as well: it spans every beam, since `--after-all` waits on all runs while a dobby sees its own, and a hub pause must leave the dobby that waits outside it.
