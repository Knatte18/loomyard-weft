# orch watcher reloads the session for a compaction hours old after a restart

`orch-watcher-stale-reload` · Note · bug, orch

Imported from [issue #390](https://github.com/Knatte18/loomyard/issues/390).

On 2026-10-05 the hub restarted a stale `lyx orch watch` (it ran a binary from 2026-10-02) with `lyx orch start --no-attach`, keeping the live orch session.
The new watcher at once ran the post-compaction reload: it typed `/scribe:prose`, `/scribe:conversation` and the role-file pointer into a session that had been working normally for hours.

The compaction it reacted to was a manual `/compact` hours before the restart.
On its first tick the watcher reads the events from the persisted cursor (`phase_events_offset`, last advanced on 2026-10-04 by the old watcher), finds that compaction through `CompactedSince`, and starts `startAutoReload` as if it had just happened.

Expected:

- A reload follows only a compaction newer than the session's last completed turn; a compaction followed by later turns needs none.
- The reload types only what a compaction loses. Claude Code re-injects the skills a session has invoked after a compaction, so typing them again is a duplicate; the role file, read through a pointer, is what is lost.
