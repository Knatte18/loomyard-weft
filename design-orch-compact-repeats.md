# Orch watcher types /compact again after a compact landed

`orch-compact-repeats` · Task · bug, orch · running · Worktree-Create

## Seen (2026-10-22, Neve)

An operator-requested compact (`cycle_requested_at` 16:52 local) compacted the orch session at 16:54, and the resume prompt reached the session.
While the resumed session was working its first turn, a second `/compact` was typed into the pane and interrupted that turn; the operator reports the watcher asking for compact several times in a row.

`lyx orch status` afterwards still read `phase: resuming`, with `state.json` holding:

- `pending_step`: `color blue` (id 142), persisted 16:54:02;
- `reload_step: -3`, `reload_skips_skills: true`;
- `stuck_by_hold: true`, `stuck: ""`;
- `compaction_baseline` 14:54:00Z, `cycle_mode: compact`.

`watch.log` has no entry for the second compact; its last line is a 14:23 local creel hold in phase `resuming`.

## Expected

Once a compact has landed and the resume prompt is delivered, the watcher leaves `resuming` and never types another `/compact` for the same request, least of all into a busy turn.
