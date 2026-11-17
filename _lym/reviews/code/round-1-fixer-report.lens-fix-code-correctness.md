```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I ran each changed path by hand against the code in context. The gate was green, but I did not rerun it.

- **`watchEditLoops` retry.** `before` is a deep copy taken after the read and before the cursor moves. On a failed persist it restores the cursor and the edit times. The next tick reads the same batch and folds it again, so the loop is found again. A nil `edits` map clones to an empty one, and `fold` copes with both.
- **`failFirstSave` row.** With the defect in place (no restore), the cursor has moved and the loop's key is deleted. The next tick reads nothing and sends no notice, so the row fails. As fixed, it notices at `dialogTick` with the offset persisted, which is what the row wants.
- **Stamp-time rows.**
  - Edits stamped 0, 120 and 240 seconds apart, read in one batch: with `signal.At` honoured, the older edits drop out of the window and no notice is sent. Ignoring `At`, all three take the run clock's time, and a notice is sent, so the row fails.
  - Edits stamped 0, 10 and 20 seconds, read at 0, 65 and 70 seconds on the run clock: with `At` honoured, the three edits fall inside the window and a notice is sent. Ignoring `At`, the first edit drops at 65 seconds, the count stays at two, and no notice is sent, so the row fails.
  - The expected offsets of three equal-length stamped edits are consistent with how the fake parser consumes stamp lines.
- **Notice-content checks.** Each part of the notice that the new checks look for appears in the `editLoopNotice` format, including the `within 1m0s` window and the delimited file path and agent id. The "no `subagent`" check for the session's own edits holds, because the notice text never contains that word otherwise.
- **`RecordEdit` open failure.** `O_CREATE` does not create parent directories, so the missing-directory row returns the wrapped `open the edit records file` error.
- **Callers.**
  - The only call sites of `RecordEdit` outside the engine are `hookedit.go` and its interface in `engine.go`, and both are updated.
  - The settings hook command and the reader both derive the file from the events file's directory with `EditRecordsFileName`.
  - A missing records file reads as empty, because `readEventsFrom` treats not-exist as no data.
  - Nothing outside the reader and the hook names the new file, so I found no cleanup list it could be missing from.

Two things I noticed and did not count as findings:

- **Skill-load edits.** The cursor no longer starts at `PromptOffset`, so edits made during a skill-load turn now count towards a loop. That offset was an `events.jsonl` offset and does not apply to the new file.
- **Unset `EventsPath`.** If `run.state.EventsPath` were ever empty, `filepath.Dir` gives `.` and the reader would look in the current directory. Every path I read sets it.

Tool calls: one `grep` for run-directory file-name users and `RecordEdit` callers outside `editrecord`. It was needed because my context had no view of other cleanup lists or implementers.
