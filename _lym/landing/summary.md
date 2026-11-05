# A Sketch is a board type of its own

An entry whose motif run ended with an approved, stored decision record now shows on the board as a Sketch: a note of the new type label `sketch`, beside `research`, `bug`, `enhancement` and `feature`.
A Sketch is not a task and never runs.
It is the stage before the triage command in `discussed-task-split`, which breaks it into tasks or a project and alone decides each one's type and recipe.

This replaces the `discussed` task status a finished Discussion-only run used to leave.

## What changes

- `boardengine` gains the built-in type label `sketch` (handle letter `S`).
  `LoadConfig` puts it first among the types unless `board.yaml` lists it, and the handle-letter check covers it.
- When a Discussion-only run finishes, `loomcli` calls the new `Board.MakeSketch`.
  The entry becomes a note with the `sketch` label and its module labels, and loses its former type and status.
  It also loses the fields only a run, a landing or the dependency graph read: `depends_on`, `isolated`, `recipe`, `profile`, `run_settings`, `pull_request` and `notify_on_land`.
- `lym board list --text` lists Sketches last, under a `sketches:` heading.
- The README renders Sketches first among the Notes.
- The `Discussed` README layer and `StatusDiscussed` are removed.
- `docs/overview.md`, the `ly:board` skill and the `set-status` help are updated to match.

## Design decisions

- **Run lock.**
  `MakeSketch` changes a task its run still holds, so `checkRunLocks` now passes exactly one net change on a held task: becoming a Sketch with no status and no other scope change (`endsAsSketch`).
  A title, body or other scope change alongside it is still refused.
- **Refused Sketch.**
  Like any write that turns an entry into a note, `MakeSketch` refuses while another entry depends on it, and the refusal matches `ErrSketchRefused`.
  `sketchFinish` then warns and leaves the entry `draft`, so the run still lets go of it and nothing starts it.
  Any other error is retried at the run's next finished transition.
- **Older boards.**
  Every board write now runs `migrateDiscussedStatus`, which turns each open task an older binary left `discussed` into a Sketch.
  A task the write would refuse, such as one another entry depends on, keeps the status as a hand-set word.
  `LegacyStatusDiscussed` stays so the start guard in `battencli` still refuses such a task, and `set-status` null still releases it.
- **Finish seam.**
  `commitStatusDeps.FinishedStatus` becomes a `Finish func() error`, because the finished write is no longer a status.

## To check by hand

- Run a motif run to its Discussion approval on a real hub and confirm the entry shows as a Sketch in the README and in `lym board list --text`, with its module labels kept.
- Confirm a board holding a task left `discussed` migrates to a Sketch on the next write.
