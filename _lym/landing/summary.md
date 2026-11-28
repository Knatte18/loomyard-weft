# Name why an orch reads as down

Issue #232 reported a running beam orch that read as down, and nothing recorded why.
Its cause is still unknown, since the hub it happened on is not available to inspect.
This change leaves the rule for what counts as live as it was and makes every down reading name its cause,
so the next occurrence shows which condition failed.
#232 stays open.
A guessed correction was left out on purpose: without evidence it risks counting dead strands as live.

## What changes

- `lym orch status` gains a `strand_down` field, empty while the strand is live.
- The orch liveness probe, which notice routing and `lym beam status` use, logs every down reading with its cause.
  The line is at Debug for an orch that never started, and at Info with the strand GUID otherwise.
- `reedengine.StrandStatus` gains `Down`, the reed-level cause for a strand that is not live.
- An orch whose recorded strand is missing from reed's state still reads as down, now as `strand-not-in-reed-state`.

| Cause | Meaning |
| --- | --- |
| `no-strand-recorded` | The orch state records no strand. |
| `no-reed-session` | The worktree has no reed session. Only the log shows this one, since `lym orch status` errors then. |
| `strand-not-in-reed-state` | The recorded strand is not in reed's state. |
| `pane-dead` | The session lists the pane as dead. |
| `pane-unlisted` | No listing carries the pane, or the strand is bound to none. |
| `pane-placement-unread` | An own-window pane is missing from the session and the server-wide lookup failed. |
| `pane-elsewhere` | The pane sits in another session of reed's server. |
| `start-shell` | The pane's start shell is in the foreground past the launch grace. |

## Design notes

- `lym orch status` and the liveness probe both take the down cause from `orchroute.ReadingOf`.
  `strand_live` and the watcher restart in `lym orch status` still come from its own strand lookup.
- A failed server-wide pane lookup used to be indistinguishable from a pane found nowhere.
  `strandsElsewhereLocked` now returns those panes separately, and its warning text changes to match.
  Reconcile ignores the new return value and plans such a pane as before.
- Notice routing does not probe the starting worktree's own orch while reed still holds its strand,
  so that path logs no down reading for it. `lym orch status` there still shows `strand_down`.

## To check by hand

- The new and updated tests run against fakes.
  Reading `strand_down` off a live reed server, for a running orch and for one whose pane was killed or moved,
  is not covered by them.

## Merge danger

**Door:** two-way
**Blast radius:** reed
