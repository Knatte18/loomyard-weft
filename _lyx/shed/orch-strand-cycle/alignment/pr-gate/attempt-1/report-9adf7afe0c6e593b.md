---
verdict: APPROVED
---

## Alignment at `pr-gate`: `orch-strand-cycle`, PR 842 at `364e81d46`

The diff against `main` (merge base `d91c50d80`, 13 card commits and a merge of `main`) covers every goal point of the board entry and the decisions D1 to D7.
No change falls outside the recorded scope.

### Goal points and decisions, checked against the diff

- Section 1 and D1: `engineAliveIDs` and `engineAliveIDSet` in `internal/reedengine/apply.go` hold condition 3 with the 30-second launch grace, the empty-command skip, the unreadable-age skip and the Windows skip.
  The pane age comes from `/proc` through cgo `sysconf(_SC_CLK_TCK)` (`proctree_linux.go`), and no `reed.json` key is added.
  Status, Resume planning, `--if-absent` and the reclaim read the judgment.
  `aliveIDSet` stays pane-level, as D1 keeps it for the Selvage and pane-option checks.
  Landing's conflict-session stop takes a present own-window shell-only strand (`internal/loomcli/landingdeps.go`), as D1's Consequences require.
- D2: `StrandStatus.Elsewhere` and `strand_elsewhere` in `lyx orch status`; reconcile and ownwindow changes keep the binding of a pane found in another session.
- Section 2 and D3: `restartWatcher` in `internal/orchcli/status.go` takes `start.lock` without waiting, re-reads under it, and reports `watcher_restarted`; `lyx orch stop` takes `start.lock`; `start` logs the restart at its existing spawn branch.
- Section 3 and D4: the note gate passes on a turn end with the note non-empty; the handoff, soft-handoff and role stencils drop the `written` reply and keep `defer`; a `written` reply is logged and ignored.
- Section 4 and D5: `lyx orch withdraw` never refuses and answers `phase`, plus `cycle_trigger` outside idle and resuming.
  This departs from the entry's "refusing once the cycle has started", and D5 records that departure and its reason, so it is a recorded decision, not a deviation.
  `cycle_request` in status, `held_by_pause` with a note on `distill` and `refresh`, the compare-and-remove under the state lock (`refresh.go` removes only its own request), and the pause discount on the stale age and handoff timeout are present.
- Folded `reclaim-dead-strand-names` and D6: `reclaimDeadNameLocked` in `internal/reedengine/strand.go` runs before `addStrandLocked`, reclaims only the asked-for name, and leaves hidden holders, holders while the pair is paused and live holders, grace included.
  It also keeps a holder that has children; that is stricter than D6 and changes no goal point.
- Folded #828 and D7: `mapDeliveryError` maps `ErrSentTextInDraft` and `ErrForeignBoxText` onto `ErrNotLanded` before `ErrBusy`; the compacting phase withdraws a still-queued `/compact` and clears the resume-noticed mark; creel's `Strands` skips a torn-down worktree with the Info and Warn split.
- Docs: the `doc.go` of reedengine, orchengine, orchcli, creelcli and shuttleengine, the orch CLI help and the claude.yaml `binary` template comment change in the same diff.
  No module is added and no PATTERN entry changes, and the decision record names none.
- Out of scope, held: the `loop-exited` halt note of #811 is untouched, and the PR body keeps #811 open; the hub orch rules section "Dead strands after a reboot (#799)" is not written, and the PR body names the `lyx orch rules` step after landing.

### Open risks carried into the decision record

The Discussion-Review carry-over (`empty-start-command-no-outcome`) is D1's recorded bound: a pane with an empty `#{pane_start_command}` keeps the #746 false-live, and the acceptance criteria and the PR body state it.
The three Plan-Review carry-overs (`card5-pane-lookup-warn-every-poll`, `card7-skip-line-repeats-per-call`, `card7-worktreetorndown-per-symbol-test`) are design details inside the decided scope; the PR body lists them under "Check by hand".
The diff answers the second one: `noteSkippedWorktree` logs a repeated skip of a path at Debug.
None of them is a scope deviation.

### Overlap

`lyx loom peers` lists no peer run and no parent commit (`own_row` 21), so there is no overlap and no landing order to give.

On the board, `batten-cancel-verb` and `plan-card-rules` depend on this task and add to it; neither duplicates or contradicts a decision here.
The note `orch-compact-repeats` (2026-10-22) describes a second `/compact` typed into a resumed busy turn, with no `watch.log` line for it.
D7's first bullet stops creel from typing a pasted inject again, which may explain the repeat.
The note's other symptom, the watcher staying in `resuming` with `stuck_by_hold: true`, is not in this run's scope, so this run does not make the note obsolete and it is not listed as superseded.

### Superseded

None.
The folded note `reclaim-dead-strand-names` is no longer on the board.
