# Fixer-report, code review round 1

## Changes

All fixes are committed in `90528d67` and the follow-up commit after it, both in this worktree and never pushed.
The fixes share files, and the round's edits went in one message, so the commits group findings instead of one per finding.

- **F1**: the edit hook's records now go to their own file, `edit-records.jsonl` beside `events.jsonl`.
  - `shuttleengine.EditRecordsFileName` declares the name once in `run.go`.
  - `settings.go` points the hook at it, and `RecordEdit` and the `hook-edit` verb take that path.
  - `watchEditLoops` reads it with its own cursor, which starts at `RunState.EditLoopNotifiedOffset` alone.
  - The cursor no longer floors at `PromptOffset`, because that offset belongs to `events.jsonl`.
  - The comments, `shuttleengine/doc.go`, `claudeengine/doc.go`, the `RunState` field comment and the summary-facing text name the new file.
  - `TestWait_EditLoopNotice` seeds and appends to the new file, and `TestBuildSettings_EditHook` expects the new path.
  - The `hook-edit` test names the edit records file too.
- **F3**: two `TestWait_EditLoopNotice` rows run with `markEvents`.
  - Three edits stamped 0, 120 and 240 seconds apart, read in one batch, want no notice.
  - Three edits stamped 0, 10 and 20 seconds, read at 0, 65 and 70 seconds on the run clock, want one notice.
- **F4**: `watchEditLoops` clones the watch before the read (`editLoopWatch.clone`) and restores it when the offset persist fails, so the next tick retries the notice.
  A new row fails the first save and wants one notice on the next tick.
- **F6**: the notice test checks each notice for the strand name, the delimited file path, the count, the window and, for a subagent row, the delimited agent id.
  It no longer compares against `editLoopNotice`'s own output.
- **F7**: a `TestRecordEdit` row puts the edit records file under a missing directory and wants an error naming `open the edit records file`.
- **F8**: `docs/overview.md` names the edit-loop notice among a run's parent notices, and carries one sentence on `hook-edit` and the `edit_loop_edits` and `edit_loop_window_s` keys.

After the gate's line-wrap findings, a third commit broke three wrapped comments at sentence boundaries:
the `hook-edit` paragraph in `claudeengine/doc.go`, the `watchEditLoops` doc in `editloop.go` and the `EditRecordsFileName` doc in `run.go`.

The gate (`lym gate test` over `./internal/shuttleengine`, `./internal/shuttleengine/claudeengine` and `./internal/shuttlecli`) passed with no failure, before and after the self-check fixes.

## Disputed

None.

## Deferred

None.
F2 and F5 are `decision`-class findings the parent held, and each ruling is under Rulings.

## Rulings

- **F2** (friction note as a notifier line): ruling #1580 records the notice as one shuttle notice line through the runner's notifier, with no file in the friction directory.
  The code already holds that, so the ruling is redundant and I changed nothing.
  I did not tell the parent, since the ruling asks for no change.
- **F5** (claude-only edit recorder): ruling #1581 records that this task builds the recorder for the claude engine only, and that a codex recorder is later work.
  The code already holds that, so the ruling is redundant and I changed nothing.
  I did not tell the parent, since the ruling asks for no change.

## Self-check

Four forks ran, and each returned a report, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-consistency, MEDIUM: a doc line in `shuttleengine/doc.go` ran over 120 characters. Fixed by splitting it at the comma.
- fix-consistency, LOW: "It also names" in `shuttleengine/doc.go` lost its referent. Fixed by rewording to "The signal also names".
- fix-consistency, LOW: `hookedit_test.go` still said "events file" and used `events.jsonl`. Fixed, and the affected-set lens's LOW finding on the same test merges into it.
- fix-affected-set, LOW: the same `hookedit_test.go` leftover. Fixed with the finding above.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
