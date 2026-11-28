# A running beam orch reads as down, and a strand missing from reed state is not covered

`beam-orch-reads-down` · Task · bug, orch, reed, cwd · landing · Finalize

## Seen

Issue #232 reported a running beam orch that lym read as down.
The landing of `beam-member-parent` (`b439f157e`) made the parent check admit every orch on the chain whether or not it reads as live,
but its commit says the cause of the down reading is not known,
and that the change does not cover the case where the orch's strand is also missing from reed's state.

## Idea

Find why a live beam orch reads as down: the liveness read, the strand's record in reed's state, or both.
Fix the read so a live orch reads as live, and make an orch whose strand is missing from reed's state read as down with that cause named.
