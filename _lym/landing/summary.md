# A Sketch is a board type of its own

An entry whose motif run ended with an approved decision record is now a Sketch.
A Sketch is a note of the built-in type `sketch`, handle letter `S`, and it never runs.
It replaces the `discussed` task status that a finished motif run used to leave.

## What changes

- **The built-in `sketch` type.**
  `boardengine.LoadConfig` puts `sketch` (`boardengine.SketchType`) ahead of board.yaml's types, with a description, unless the file lists it already.
  Every hub accepts the label with no config edit, and `lym board labels` lists it.
  A board.yaml type starting with `S` is now refused by the existing handle-letter check, which names the label to rename.
- **`Board.MakeSketch`.**
  In one write, the entry becomes a note.
  Its labels become `sketch` followed by its module labels, and its former type labels are dropped.
  It loses its status, `depends_on`, `isolated`, `recipe`, `profile`, `run_settings`, `pull_request` and `notify_on_land`.
  Its title, brief, body, project, issues, dates and discussed flag (with its `Sketch` link to the decision record) stay.
  It writes nothing for an absent, done or already sketched entry.
  Like any write that makes an entry a note, it refuses while another entry depends on the entry.
- **Run lock.**
  `checkRunLocks` lets a held task through when it becomes a Sketch with no status (`endsAsSketch`).
  The entry leaves its run in the same write, so it never reads as a released task in between.
  The board replay runs the same check, so a replayed Sketch write is not refused.
- **The motif run's finish.**
  In `loomcli`, the commit-status seam's `FinishedStatus` string becomes a `Finish` callback.
  For a recipe with no Finalize row (`finishesAsSketch`), `Finish` calls `MakeSketch` on the run's entry.
  A failed `Finish` is retried at the next finished transition, as a failed status write already was.
  When the board refuses the Sketch, the run warns and sets the entry to `draft`, so the run still releases the entry and nothing starts it.
- **The `discussed` status is retired.**
  These are removed:
  - `boardengine.StatusDiscussed`
  - the `Discussed` layer and README subsection
  - the `discussed` title code span
  - batten's start-guard branch for the status
  - the help text that described releasing a discussed task
  
  At the start of its next write, the board turns a task an older binary left `discussed` into a Sketch (`migrateDiscussedStatus`).
- **Listing.**
  In the README, Sketches come first among the Notes, under a `Sketches` heading, with research next.
  `lym board list --text` and `find --text` print Sketches after every other entry, under a `sketches:` line.
- Docs updated:
  - the `boardengine` package doc (new `# Sketch` section, Labels, Handles, README, Run lock, Draft and Migration)
  - the `boardcli` and `loomcli` package docs
  - `docs/overview.md`
  - the board skill

## Design decisions

- **`sketch` is built in, not a template entry.**
  `types` is an open map that reconcile carries whole from each hub, so a template entry would never reach existing hubs.
  lym writes the label itself, so the label must exist wherever the motif recipe runs.
- **`sketch` goes into `Config.Types` in `LoadConfig`, not into `Vocabulary()`/`Outputs()`.**
  This keeps the existing handle-letter check covering it.
  It also leaves a path-only `Config`'s vocabulary empty, as before.
- **The run-lock exemption is a rule on the net change, not a marker that only `MakeSketch` carries.**
  The replay can judge the same change, and the critical section needs no new result type.
  A hand upsert that makes the same change on a held task also passes, as a hand `set-status` already does.
- **The Sketch drops the run and dependency fields.**
  Triage gives each resulting task its own type, recipe and profile.
  A note cannot carry `depends_on` or `notify_on_land`.
- **The `discussed` status is migrated, not kept.**
  Nothing writes it any more, so a migration leaves no dead layer behind.
  A discussed task that another entry depends on keeps the status as a plain hand-set word.

## To check by hand

- Release a motif task live and watch the entry through approval.
  It should end as an `S` note under `Sketches` in the README, keep its module labels and keep the `Sketch` link to its decision record.
  Nothing here ran a real motif task end to end.
- If any hub has a board.yaml type starting with `S`, its board config now refuses to load until that type is renamed.
- The fallback to `draft` when `MakeSketch` is refused is covered only by reading the code.
  No test drives a motif finish on an entry that something depends on.
