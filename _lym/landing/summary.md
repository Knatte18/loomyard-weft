# Run agents wait for their parent's reply before their run ends

A run agent that asked its parent with `lym creel post --parent --expect-reply` and then ended its turn, as the parent directive tells it to, could have its run finalized at that same turn end.
That happens to an ungated run whose output files already exist, or to a gated run whose gate passes.
The strand was then removed, so the orch's answer came back from creel as returned.
The drift judge had a second problem: its autonomous mode text gave it no route for a question, so it wrote one into its pane, where nothing reaches the orch.

## What changes

**creel: `awaits-reply`.**
A new protocol verb and `Store.AwaitsReply(strand)` report whether a strand is waiting on an answer.
That is true while a strand holds a message it sent with `--expect-reply` that is neither answered nor settled unanswered.
It is also true while the asker holds an unread reply to one of its messages, so a reply already given but not yet read still counts.
A question a program holds is not counted, since creel never chases it and so it might never settle.
Every counted question does settle, through a reply, the chases or the holder's removal.
`GoClient.AwaitsReply` sends the verb, and `creelcli.AwaitsReplyProbe(location, program)` wraps it as a probe that runs as a program.

**shuttle: the reply hold.**
`Runner.SetReplyProbe` gives `Wait` a probe (`internal/shuttleengine/replyhold.go`).
`Wait` asks it at any turn end that would finish the run: an ungated Done, a gated Done before its gate runs, or a gated arrival after a pause lifts.
While the run's strand awaits a reply, the turn end is held: nothing is finalized, no gate is run and no re-prompt is sent.
Any event or turn start read after that ends the hold.
So the reply, which creel's idle waiter delivers, starts the agent's next turn, and that turn's end decides again.
If the question settles without a reply, two quiet probes `gateRecheckEvery` apart release the hold.
The run then finalizes done, or evaluates its gate as it would have at that turn end.
A probe error, a runner that has no probe, and a run whose record names no strand all behave as before.
The hold sends no notice and never extends the run deadline.

**Wiring.**
The runners that already send held-turn-end notices to a parent now also get the probe.
Those are the runners of loom, burler, webster and batten, which covers darn, the drift judge, the gate fixers, the tackler and the arbiter.

**Stencils and mode text.**
The parent directive now says the run waits for the answer, which arrives as `[creel]` mail and starts the next turn.
The operator-ban line now says a question in the pane reaches no one.
The drift judge's autonomous mode text sends any question it cannot settle to its parent as creel mail.

## Design decisions

- The fix holds the turn end in shuttle instead of asking agents to run a foreground `--wait-reply`.
  A foreground wait would block the session against tool timeouts and the keep-warm cap, and it would depend on every agent following the instruction.
  With the hold, the existing directive ("then end your turn") becomes true for every role at once.
- The release waits for two quiet probes, not one.
  A reply can be read in the instant between a probe and the turn start that reply causes.
  The second probe, five seconds later, sees that turn start first.

## Bounds

- A question still unread by the orch holds the run until the orch reads it or the run deadline passes.
- Runners that have no notifier (weaver's own CLI, tender, the gate card fixer, orchcli) get no probe and behave as before.
- Questions sent to another hub's strand live in creel's outbound map and are not counted.

## Tests

- `TestStore_AwaitsReply` (creelengine) covers an open question, an unread reply, a read reply, a plain message, a question a program holds, another strand's question, a question settled unanswered and an empty strand.
  `TestDaemon_AwaitsReplyAnswersThroughTheProtocol` sends the verb through the daemon.
  A creelcli subtest posts a question with `lym creel post --expect-reply` and asks `AwaitsReplyProbe` about the asker and the receiver.
- `TestRun_Wait_ReplyHold` (shuttleengine) checks four things.
  A reply's turn ends an ungated or gated run only at the turn end after the reply, and a gate is not run while the hold lasts.
  A question that settles without a reply ends the hold after two quiet probes.
  A probe error ends the run at once.
  A run with no strand name is never probed.

## Merge danger

**Door:** two-way
**Blast radius:** shuttle
