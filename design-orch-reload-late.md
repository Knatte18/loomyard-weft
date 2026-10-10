# orch watcher sends the post-compact reload minutes late while the operator chats

`orch-reload-late` · Note · bug, orch

Imported from [issue #481](https://github.com/Knatte18/loomyard/issues/481).

After a distill on 2026-10-10, the watcher's resume steps (`/color`, `/reload-plugins`, the "read role.md and the orch note" prompt) reached the orch session almost three minutes after the compaction, after the operator had already held four exchanges with the un-reloaded session.
The operator reports the same lateness after earlier compactions as well; `lifecycle-waits` (done) and #356, #390 fixed neighbouring cases, not this one.

## Timeline (UTC, from the orch transcript and `.lyx/orch/watch.log`)

- 04:07:10 handoff note requested (`handoff-20261010T040710Z.md`), written, turn ended.
- 04:08:25 compaction boundary written to the transcript.
- 04:08:45 the operator's next message arrives; the session works on it with no role, skills or color reloaded.
- 04:08:26, 04:09:53, 04:10:24, 04:10:57: at each of the orch's turn ends the watcher logs `orch: session state disagrees with the idle probe` with `idle=false state=asking cause=awaiting-input`, and injects nothing.
- 04:11:09 the phase enters `resuming` (`state.json`), and 04:11:12 `/reload-plugins` and the resume prompt land.

So the session's own state said it was waiting for input at every turn end, but the pane idle probe said not idle, and the watcher waited for the probe.

## Expected

- The resume steps follow the compaction boundary at once, before the operator's next message is taken, or are queued so they run ahead of it.
- A turn end whose session state is `asking`/awaiting input is enough to inject; a disagreeing pane probe does not hold the reload back for minutes.
- The watcher logs why each resume step waited, so the delay can be read from `watch.log` without the transcript.
