# Count only repeat edits toward an edit loop

The edit-loop watch counted every edit of one file by one agent.
A fixer applying several review findings to one file, or a writer whose whole output is one document, edits that file many times while making progress, so the watch sent the parent notices for ordinary work.
The hub's shuttle config set `edit_loop_edits: 0` to silence them.

Now an edit counts toward a loop only when it leaves the file with content an earlier edit of it by the same agent left within `edit_loop_window_s`.
An agent that makes progress leaves new content each time and never counts, however often it edits the file.
The template's `edit_loop_edits` is 3, now counting repeats.

## Design

- `RecordEdit` in `claudeengine` appends the SHA-256 of the edited file's content, read by the hook right after the edit, to each edit record under the key `content_hash`.
  A file the hook cannot read is recorded with no hash.
- `editLoopWatch.fold` in `shuttleengine` keeps per agent and file the recent contents and the repeat times, and prunes both by the window.
  An edit whose signal carries no hash never counts.
  A loop is complete at `edit_loop_edits` repeats, then that file's history restarts.
- The notice now says the file was edited back to content it held earlier, with the count of repeats.
- A new config reinterpretation, `shuttle-edit-loop-repeats` (`DropEditLoopEdits`, registered in `configreg`), carries a hub's `shuttle.yaml` across the changed meaning of the key.
  A `0`, or a null or empty value, is dropped so the template's count applies; the rewrite reports it and names the `lym config shuttle` command that sets another count.
  Any other value is kept and flagged, since a count of every edit has no equivalent count of repeats.
- `docs/overview.md`, the `edit_loop_edits` docs and the `hook-edit` help text describe the new meaning.

## To check by hand

- `DropEditLoopEdits` in `internal/shuttleengine/reinterpret.go`: confirm that keeping and flagging a non-zero `edit_loop_edits`, rather than rewriting or dropping it, is the intended handling.
- Window pruning in `editLoopWatch.fold` in `internal/shuttleengine/editloop.go`: confirm that `editloop_test.go` covers pruning of old contents and old repeats, including a repeat that falls out of the window between two edits.
- Not run: the regression check did not run for this task, and no live run of a fixer against the new watch was made.
