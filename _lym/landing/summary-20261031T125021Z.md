# Dobby holds a slug with a finished run record and beam status says why

A released task whose slug already had a `done` run record under the prime was never started and never reported.
Dobby spawned a first pass about every 30 seconds for over an hour, `lym beam status` showed `ready to start` with no reason, and the orch got its notice only after `lym batten cleanup --apply`.

## What changes

- `battencli.DobbyTasks` reads each released task's run status under the prime.
  When it reads `done`, the task's new `TaskView.StartRefusal` carries batten's done-slug refusal, the same text a hand `lym batten run` prints, with its way forward: `lym batten cleanup --apply` from the prime, the leftover task branch removed, or a new slug.
  A status file that does not decode holds nothing and logs a warning, as the run listing does.
- `doneSlugRefusal` in `internal/battencli/arm.go` becomes a package function of the prime location and slug, so the pre-run hooks and `DobbyTasks` share one text.
- In dobby's start step, a released task with a start refusal is a hold, like a pending dependency.
  Dobby spawns no pass for it.
  It records the refusal in the task's start record with no backoff and posts the usual `<slug>: dobby could not start the task: <text>` notice once per text, and again when the board entry changes.
  The record is read every tick, so the task starts on the first tick after the record is gone, with no hour of backoff left after a cleanup.
- `BuildView` carries the hold as `WaitingTask.StartRefusal`, and `Ready` treats it as a hold.
- The dobby lines in `lym beam status` and the beam window render a held task as `waits on a start batten refuses: <text>`.
  A ready task whose refused start's backoff has run out now renders `ready to start, last refused start: <text>` instead of a bare `ready to start`.
- The `dobbyengine` and `battencli` package docs describe the hold and the new lines.

## Why the notice came late

Two things combined.
The old record's status file made every failed first pass read as a claimed run in `firstPassExited`, which cleared the start backoff and handed the exit to the run's failing-pass count instead of the refused-start path.
A done run is then dropped from dobby's run list on the next tick, and its state entry with it, so the failing count was reset to one every pass and never reached the notice threshold of three.
The only notice went out when `lym batten cleanup --apply` removed the record while a pass was in flight: that pass exited with no status file left, so it read as a refused start and noticed at once.
Creel did not hold the notice.
With the hold, dobby never spawns a pass for a finished slug, so this path is no longer reached.

## Where the other machine's record came from

The slug had run before.
Board task 755, `claude-md-lym-rules-out`, was started by Neve's prime dobby on 2026-10-28 at 15:05Z, landed, and its run went `done` at 15:24Z; the landing pruned the entry from the board.
On 2026-10-29 a new note, entry 903, was filed under the same slug, and was later promoted and released.
Run records are tracked under `_lym/shed/<slug>/` in the prime, so the finished record reached the other machine with the prime and collided with the new entry.

## Design decisions

- The finished record is handled where dobby starts the task, not where the board entry is created or released: the board engine reads no run records, and a record can arrive with a prime sync after the entry was released.
- The hold has no backoff: reading one status file per released task each tick is cheap beside the board and run reads dobby already makes, and it lets the task start right after a cleanup.
- Only the `done` state holds, the one state batten refuses as finished; every other state is a run batten resumes and is started as before.

## Tests

- `TestReadyToStart` gains a row for a task with a start refusal.
- `TestStartReady_RefusalsAndBackoff` gains a case: a held task spawns no pass, notices once, records no backoff, notices again after an entry change, and starts on the tick after the refusal is gone.
- `TestBuildView` and `TestDobbyLines` gain rows for the held task and for a ready task with an expired refused start.
- `TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord` covers no record, a `done` record, a running record and an undecodable record.
