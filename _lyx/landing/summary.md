# Orch lifecycle: dead strands read dead, watcher restarts, cycle withdraw

The orch lifecycle now holds up when a session, its watcher or the whole machine goes away.
A strand whose engine exited reads dead and frees its name, a dead watcher comes back, and the context cycle needs no reply, can be withdrawn and never re-sends a delivered `/compact`.

Closes #746
Closes #799
Closes #802
Closes #828

#811 is only partly fixed here (the name clash).
Its `loop-exited` halt-note part belongs to task `loop-exit-halt-names-its-cause`, so #811 stays open until both land.

## What changes

- **Strand liveness (`internal/reedengine`).**
  A strand is live only when its pane is listed and not dead, sits in this worktree's reed session, and its foreground command is not the shell the pane started with.
  Every reader gets this: `lyx orch status`, `lyx orch start`, webster's `StrandLive`, loom's driver check, creel's `Strand.Live`, shuttle's wait and attach, reed's directory rows and Resume planning.
  `lyx orch start --continue` then resumes a strand whose engine exited in one step.
  Both pane listings gain `#{pane_current_command}` and `#{pane_start_command}`.
- **Pane elsewhere.**
  A strand whose pane exists only in another session of reed's server reads dead, keeps its binding through reconcile, and `lyx orch status` names the placement as `strand_elsewhere`.
  Removal never kills such a pane.
- **Watcher restart.**
  `lyx orch status` restarts a dead watcher for a live orch strand and reports `watcher_restarted: true`; `lyx orch start` does so for a live strand too.
  Both log `orch: restarted a dead watcher`.
  `status` takes `start.lock` without waiting, and `lyx orch stop` now takes it.
- **Note gate without a reply.**
  The note phase ends at a turn end once the requested note file exists and is not empty.
  The handoff and role stencils stop asking for `written`; a `written` reply is logged and ignored.
  `defer` to a soft request still declines the cycle while the note is unwritten.
- **Withdraw and visibility.**
  `lyx orch withdraw` removes a pending `distill` or `refresh` request in any phase.
  `lyx orch status` gains `cycle_request` with its mode, time and `held_by_pause`; `distill` and `refresh` say when a hub pause holds the request.
  Every writer and remover of `cycle-request` now compares and removes under the orch state lock.
  Hub pause time no longer counts toward a request's stale age or the handoff timeout, so a request made under a pause runs after the resume.
- **Dead names.**
  A reed add removes a dead, non-hidden holder of the exact asked-for name before numbering, unless the pair is paused.
  A resumed row or a fix attempt keeps its canonical name.
- **#828.**
  Creel maps `ErrSentTextInDraft` and `ErrForeignBoxText` to `ErrNotLanded`, so a pasted inject is never typed again.
  The watcher withdraws a still-queued `/compact` once it sees the compaction.
  Creel's `Strands` skips a worktree torn down mid-lookup.
- The claude.yaml `binary` key's template comment now says the binary must be the engine or `exec` it.
  The `doc.go` of each changed package and the orch CLI help are updated.

## Design decisions to review

- Condition 3 tests "not the shell", not "is `claude`", since reed holds no provider knowledge (`PATTERN-shuttle-provider-seam`).
  A pane where the operator started another program after the engine exited reads live.
- Condition 2 checks the reed session, not the exact window, so a pane moved between windows of the session still reads live.
- A 30 second launch grace, read from the pane's first-process age in `/proc` through cgo, skips condition 3 for a young pane.
  No launch time goes on the strand, so no `reed.json` key is added.
- Condition 3 is skipped on Windows, and for a pane whose `#{pane_start_command}` is empty (a session booted without the `default-command` pin), which keeps the #746 false-live.
- A shell-only strand now reads dead past the grace, so every removal edge kills its bare-shell pane: reclaim, `start`'s relaunch, Resume's relaunch, and landing's conflict-session stop.
  Resume brings back an engine the operator exited on purpose.
- `lyx orch withdraw` never refuses.
  It departs from the board entry, which asked for a refusal once the cycle has started: it answers `withdrawn: false` with the phase, and `cycle_trigger` outside the idle and resuming phases.
- A name reclaim leaves hidden holders, holders inside the grace, numbered forms and paired-pause strands alone.

## Check by hand

- Residual cases where a live engine reads dead after the grace: an engine started through a shell wrapper that does not `exec`, one suspended with Ctrl-Z, and one that makes a child shell the tty's foreground group.
  Check against a real Claude Code under tmux.
- An rc file that `exec`s another shell leaves the strand reading live after the engine exits.
- Condition 3 is unverified on Windows, where it is skipped.
- Reed's server is shared by the hub's worktrees, so after a reboot a stale binding can show a sibling worktree's pane placement in `strand_elsewhere`.
- Findings left open by plan review, each to check in the code:
  - the pane lookup for a pane absent from the session (reed `ownwindow.go`) may warn on every poll;
  - the creel `Strands` skip line may repeat on every call;
  - the torn-down-worktree skip in creel's `Strands` may lack a test for each symbol that can fail after a successful resolve.
- After landing, the orch removes the hub orch rules section "Dead strands after a reboot (#799)" with `lyx orch rules`; a task's code change never writes Bolt.
