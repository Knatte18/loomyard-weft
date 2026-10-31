# Dobby holds a task whose slug has a finished run record

A released task whose slug already has a finished run record failed every dobby pass for an hour.
`lym beam status` showed it as `ready to start` with no reason, and the orch heard of the failure only after the record was removed by hand.
Dobby now holds such a task before spawning a pass, tells the orch once, and `lym beam status` says why the task is not started.

## What changes

- `battencli.DobbyTasks` reads the run status of each released task's slug under the prime.
  When its state is `done`, the task carries the refusal a `lym batten run` of that slug gives: the done-slug text with its way forward (`lym batten cleanup --apply` from the prime, the task branch removed, or a new slug).
  An unreadable status holds nothing and logs a warning; any state other than `done` is a run batten resumes and starts as before.
- The start step in `internal/dobbyengine` spawns no pass for a task that carries this refusal.
  The hold applies only to a task dobby would otherwise start: one owned elsewhere, with open dependencies or covered by a pause is left to those rules.
  The record is read every tick, so the task starts on the first tick after the record is gone, with no backoff to wait out.
- The hold posts the `dobby could not start the task` notice once per refusal text, on the first tick that sees it, and again when the board entry changes.
  It is recorded in the existing start record with no backoff, so the state record gains no field.
- `lym beam status` and the beam window show a held task as `waits on a start batten refuses: <text>`.
  A ready task whose refused start's backoff has run out shows `ready to start, last refused start: <text>`, so `ready to start` has no reason only for a task never refused.
  The `lym beam status` help text names both.
- `doneSlugRefusal` in `internal/battencli` is now a function of the location and slug, so dobby and the batten pre-run hooks give the same text.

No issue is closed.

## Design decisions

- The hold is a dependency-like state decided by the board-reading step, not a failure handled after a pass.
  A pass of a done slug fails before it claims anything, and the leftover status file made dobby read each failed first pass as a claimed run, which cleared the backoff and spawned a new pass every tick.
  Holding before the spawn removes that path.
- Refusal recording is shared between a refused start and the hold (`recordRefusal` in `internal/dobbyengine/starts.go`); they differ only in how the retry is scheduled.
- Out of scope: refusing a board entry at creation or promotion for a slug with a finished record (the board reads no run records), and changes to `lym batten cleanup` or the refusal text.

## Why the notice came late

A done run is dropped from dobby's run list every tick, with its state entry.
The failing-pass count of the misread first pass therefore never reached the notice threshold.
The only notice came when `lym batten cleanup --apply` removed the record while a pass was in flight, so that pass read as a refused start.
Creel did not hold or delay the mail.
The hold removes the misreading path, so no further change to the notice threshold was made.

## Why a record of another machine existed

The board history shows an earlier task with the same slug ran from Neve and landed on 2026-10-28, and a new entry was filed under the same slug the next day.
The run record is tracked under `_lym/`, so it reached this hub's prime with the repository.
Why that record read as `done` here was not further determined.

## To check by hand

Not run: a live dobby against a real finished record.
To check, release a task whose slug has a `done` run record under the prime, then confirm that `lym beam status` shows `waits on a start batten refuses`, that one notice reaches the orch, that `.lym/beam/dobby.log` shows no repeated passes, and that the task starts on the next tick after `lym batten cleanup --apply`.
