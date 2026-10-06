# orch watcher: stale reload after restart, and a restart race

`orch-watcher-stale-reload` · Note · bug, orch

Imported from [issue #390](https://github.com/Knatte18/loomyard/issues/390).

On 2026-10-05 the hub restarted a stale `lyx orch watch` (it ran a binary from 2026-10-02) with `lyx orch start --no-attach`, keeping the live orch session.
The new watcher at once ran the post-compaction reload: it typed `/scribe:prose`, `/scribe:conversation` and the role-file pointer into a session that had been working normally for hours.

The compaction it reacted to was a manual `/compact` hours before the restart.
On its first tick the watcher reads the events from the persisted cursor (`phase_events_offset`, last advanced on 2026-10-04 by the old watcher), finds that compaction through `CompactedSince`, and starts `startAutoReload` as if it had just happened.

Expected:

- A reload follows only a compaction newer than the session's last completed turn; a compaction followed by later turns needs none.
- The reload types only what a compaction loses. Claude Code re-injects the skills a session has invoked after a compaction, so typing them again is a duplicate; the role file, read through a pointer, is what is lost.


## From issue #393

Imported from [issue #393](https://github.com/Knatte18/loomyard/issues/393).

The orch session cannot load `ly:board`, though the role procedure (`.lyx/orch/role.md`) and the orch watcher's reload both point to it.

Seen 2026-10-05 in the hub's orch session, after deploying `c140db21a` with `update-plugins.sh`:

- `~/.claude/plugins/cache/loomyard/ly/1.0.0/skills/board/SKILL.md` exists, and the prime's `.claude/settings.json` enables `ly@loomyard`.
- The Skill tool answers `Unknown skill: ly:board`, both before and after the watcher's role reload.
- The session's Claude Code process started on 2026-10-03, and `lyx orch` keeps it alive across context cycles through `/compact` and a typed reload.

Claude Code reads the skill list when the process starts.
A skill added or renamed by a later deploy stays unknown to a long-lived session, and so does every procedure that names it.

Possible fixes:

1. A context cycle, or the first cycle after a deploy, restarts the Claude Code process (`claude --resume` or a fresh session with the orch note) instead of compacting in place.
2. `update-plugins.sh` tells the running orch that skills changed, and the watcher schedules a restart.
3. The reload message checks that each skill it names loads, and reports a missing one to the operator instead of failing silently in the session.

## Restart race (2026-10-06)

`kill <pid of lyx orch watch>` followed at once by `lyx orch start --no-attach` answered `attach-only` and spawned no watcher, since the old watcher still read as live.
A second `lyx orch start --no-attach` a moment later spawned it.
`start` should wait for a watcher it sees dying, or the restart needs its own verb.
