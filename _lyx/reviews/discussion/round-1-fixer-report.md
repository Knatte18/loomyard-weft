# Fixer report — discussion round 1

Every finding's premise was checked against the code at HEAD, and against tmux's source for F8, before the change was made.

## Changes

- F1 (BLOCKING), fixed as proposed: D1's Bound of condition 3 names the misjudged-live cases, a non-exec wrapper and a Ctrl-Z suspended engine, and why they read as the shell.
  The wrapper case is bounded by a new Constraint, that the claude.yaml `binary` key must be the engine or `exec` it, with that key's template comment added to Scope's docs.
  The suspended case is a stated residual.
  D1's Consequences and D6's Bound state that every removal reading D1 liveness kills a present bare-shell pane, and that a misread pane is killed with its engine.
- F2 (MEDIUM), fixed as proposed: D2's Rationale states today's guard (`killStrandPanes` with `paneIDsInSession`, `killReplacedOwnWindowPaneLocked`) and keeps as new only the lookup, the placement and `strand_elsewhere`.
  D6 cites removal's existing session-membership guard, and the Scope bullet no longer presents the guard as new.
- F3 (MEDIUM), fixed as proposed: D1 condition 2 reads every strand's pane from the session-wide listing.
  D2 defines elsewhere as any session other than the reed session.
  A new acceptance criterion covers a shared-window pane moved within the session.
- F4 (MEDIUM), fixed as proposed: D5's withdraw removes a pending marker in any phase and never refuses.
  It never reads a stale `CycleTrigger`, and the criteria cover a refresh during a requested cycle and a withdraw during an auto-compaction reload.
  The `PATTERN-refusal-way-forward` constraint line is dropped, since withdraw has no refusal left.
- F5 (MEDIUM), fixed as proposed: every watcher removal of the marker (the cycle start of every trigger, the stale removal and `removeSatisfiedRequest`) is one locked compare-and-remove.
  A hard or soft start consumes a request pending at its read and leaves a newer one, and a new criterion covers a refresh written during a hard start.
- F6 (MEDIUM), fixed as proposed, first option: the stale age excludes time a hub pause entry covered the request, read through `pauserecord.Record.CoverageOf` with an empty slug and bounded by `pauserecord.Retention`, so the `held_by_pause` note holds.
  A criterion covers a pause longer than `handoff_timeout_s`.
- F7 (MEDIUM), fixed as proposed, first option: the grace reads the age of the pane's first process (`#{pane_pid}`, POSIX `ps -o etime=`), a fact every binary leaves because reed never respawns a pane.
  No launch-time key is added, and the record cites internal/state/doc.go's unknown-key rule.
  The `PATTERN-tolerant-records` constraint and the criterion about strands recorded before the change are updated to match.
- F8 (LOW), fixed as proposed: tmux 3.4 and 3.6 `format_cb_current_command` falls back to the pane's argv and then its shell.
  Reed pins `default-command` to the shell, so a failed read answers the shell.
  The Constraints fact and D1's bound record this, with the Linux `osdep_get_name` failure cases and the residual of a read that races the group's exit.
- F9 (LOW), fixed differently: the proposal offered the session's pinned `default-shell` or `cfg.Shell` as the comparand.
  Neither is sound, since internal/reedengine/doc.go says the attach pre-flight re-pins `default-shell` and `default-command` to the currently resolved shell on every attach, so both follow a changed key.
  Condition 3 compares against the pane's own `#{pane_start_command}`, frozen at spawn.
  The rc-exec false-live case is added to Open risks.
- F10 (LOW), fixed as proposed: D1's Consequences and the matching criterion include landing's `conflictSessionStopper`.
- F11 (LOW), fixed as proposed: D6's Bound states that a holder inside its launch grace keeps the name.
- F12 (LOW), fixed as proposed: D7 and its criterion skip a worktree whose reed state read, wire or status step fails when its directory is gone afterwards.
- F13 (LOW), fixed as proposed: the shell-only criteria are qualified as POSIX, and the Windows reading is stated.
- F14 (LOW), fixed as proposed: the open risk carries the evidence, Claude Code 2.1.308 under tmux 3.6 with `ps -t`, which the fixer reproduced on its own pane.
- F15 (NIT), fixed as proposed: the Node and version-named sentence is dropped from D1's departure, which points to support-log.md's Rejected alternatives.

## Self-check

- fix-consistency LOW, support-log Q2 ledger still names a recorded launch time: fixed as proposed (ledger line updated).
- fix-consistency LOW, support-log Q6 ledger scopes the lock to the cycle start only: fixed as proposed (ledger line updated).
- fix-way-forward MEDIUM, the hard-start criterion ignores `removeSatisfiedRequest` removing a same-mode refresh during a clear: fixed as proposed (criterion names both outcomes).
- fix-way-forward MEDIUM, `withdrawn: false` reports a `cycle_trigger` that can be stale: fixed as proposed (reported only outside the idle and resuming phases, in D5 and its criterion).
- fix-way-forward LOW, no outcome stated for an unreadable pane age: fixed as proposed (an unreadable age counts as inside the grace).
- fix-claims MEDIUM, the attach pre-flight re-pins the shell, so the session pin is not a stable comparand: fixed as proposed (condition 3 compares against the pane's `#{pane_start_command}`).
- fix-proposal MEDIUM, reconcile still clears a moved shared-window strand's binding, so it would relaunch: fixed with own change (D1 condition 2 has reconcile keep that binding and render leave the pane in place); the fork named no specific change.
- fix-proposal LOW, widening `presentPanes` reaches pane-level readers unstated: fixed as proposed (D1 condition 2 names them).

## Disputed

None.

## Deferred

None.
