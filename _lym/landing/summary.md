# Hold a run's turn end while its agent awaits a creel reply

A run agent that asked its parent with `lym creel post --parent --expect-reply` and ended its turn could finish the run before the reply arrived.
Creel then returned the reply as undelivered, and the run went on without the answer.
The drift judge, told nothing about creel mail, asked in its own pane, where no notice reaches the orch.

Now a run holds a turn end that would finish it, or start its gate's evaluation, while the run's strand awaits a reply.
The reply starts the agent's next turn in the live session, and that turn's end decides afresh.
The drift judge's autonomous mode now sends a question it cannot settle to its parent as creel mail.

Closes #186

## What changes

- `creelengine` gains an `awaits-reply` verb, `Store.AwaitsReply`.
  It is true while the strand has an open question, or holds an unread reply to one of its own messages.
  A question held by a program is never chased, so it does not count.
- `creelcli` gains `AwaitsReplyProbe`, a probe over the Go client.
  A caller in a non-orch strand pane signs as that strand, and falls back to signing as the program when the daemon refuses the claim as coming from outside the pane.
- `shuttleengine` gains `Runner.SetReplyProbe`, `Runner.HasReplyProbe` and `replyhold.go`.
  `Wait` holds a Done turn end, gated or not, while the probe reports a reply awaited, and sends no parent notice for the hold.
  The hold never extends the run deadline.
  Any event or turn start read afterwards ends it.
- The loom, burler, webster, batten and card-fixer runners set the probe.
  The card fixer signs as `GATE`.
- The `parent-directive-parent` and `parent-directive-operator-ban` stencils say that the run waits for the answer, and that nothing written in a pane reaches the parent.
- `docs/overview.md` and the `creelengine`, `creelcli` and `shuttleengine` package docs describe the probe and the hold.

## Design decisions

- The hold releases only after two probes in a row, `gateRecheckEvery` apart, find no reply awaited, and the agent's pane reads idle.
  The quiet probes alone could end the run mid-turn: a reply read between two probes starts a turn that may write no turn start.
  A busy pane keeps the hold; the next interval reads it again.
- A probe error reads as no reply awaited, so a run ends as it would with no probe.
  It logs one warning per episode.
- The sweep now also visits a strand that sent a question still open, so the asker's removal settles the question unanswered.
  A later strand reusing the name inherits none.
  A question from another hub or machine is never the sweep's to settle.
- A runner with no probe, the default, holds nothing, so existing callers keep their behaviour.

## Merge danger

**Door:** two-way
**Blast radius:** shuttle
