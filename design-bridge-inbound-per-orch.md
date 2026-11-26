# Cross-machine mail into the receiving orch own channel instead of a channel per orch pair

`bridge-inbound-per-orch` · Note · enhancement, slack, creel

## Seen (operator, 2026-11-26, relayed by norce:orch on osl46)

The first cross-machine test over the Slack bridge worked both ways (Neve #2650, osl46 #129).
Setup was clumsy: `approve <machine>` by name failed, and each side needed `approve <machine> <app id>`.
Bridge channels are made per pair of orchs, so their number grows with the square of the orchs and machines.

## Idea

One inbound channel per orch: its existing channel.
An orch on another machine posts its mail into the receiver's channel.
The receiving relay checks the sender bot's app id against the approved peers, as today.
Channels then grow linearly.

Cost and open points:

- A conversation between A and B is split across two channels.
  A could mirror its outgoing mail into its own channel, or the two could use threads, so the operator sees the whole exchange in one place.
- The relay must keep telling the operator's messages apart from peer bots' posts in the same channel.
- Why approve by name found no bot `lym-neve` is unexplained; a design that still names peers must find them.

The operator will think further about it; this is a note, not a task.
