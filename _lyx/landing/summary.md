# A run's agent can file a note, never create or promote a task
A run's agent session can no longer create or promote a task on the board.
`gate-runs-once` filed a task with `lyx board upsert`, the task depended on it, the Land notice named it "newly ready", and the orch started it as a full loom run the operator never approved.
Only the operator decides what becomes a task, and with which recipe and profile; a run may at most file a follow-up as a note, which waits on the board.

## What changes

- `internal/boardengine`: the write critical section refuses a board write whose net change leaves a task whose slug was not a task before.
  A new task, a promote, a note upserted to `"kind":"task"`, an `upsert-batch` carrying a new task and a `merge` whose upserted entry is a new task all count, through every verb.
  Notes, edits of an existing task and renames are unaffected.
  The refusal names the session and the entry, and says to file the follow-up as a note with `"kind":"note"`.
- `internal/boardcli`: every write verb's board carries the caller's `LYX_STRAND_NAME` through the new `boardengine.Config` field `StrandName`.
- Stencils: the Discussion, Plan and Darn writer templates say a follow-up becomes a note, never a task, and never promotes one; the discussion and plan review rubrics flag a decision or card that files or promotes a task.
- Docs: `internal/boardengine/doc.go`, `internal/boardcli/doc.go` and the `ly:board` skill describe the refusal.

## Design decisions

- One net-change check, modelled on `checkRunLocks`, instead of a guard per verb.
  A rename is judged with the loaded slug renamed, so it creates no task.
- A run's agent is a strand name carrying a task slug under any role but the orch's, matched with `agentname.MatchesRole` so a numbered orch role such as `orch-2` is exempt.
  An empty name (the operator at a terminal), a slug-free name (the hub orch) and an orch thread in a task worktree write as before.
  An orch thread acts for the operator.
- The strand name is read in `boardcli`, never in `boardengine`.
- The guard reads only the name the caller passes: it stops a mistaken write from a run's agent, not one that clears its environment.
- A `merge` from a run's agent whose upserted slug is a new task is refused even when it folds existing tasks.
- The orch keeps starting each task named after `newly ready:` at once.
  Once this guard lands, only the operator creates tasks, so every task on the board is already approved.

## Not covered

A Go producer that writes the board in-process is not covered, since only the CLI sets the strand name.
No producer creates a task today.

## To check by hand

Nothing was run by hand; the `boardengine` and `boardcli` tests cover the guard.

- `internal/boardengine/taskguard.go:14`: confirm a numbered orch thread (`<shortname>:<slug>:orch-2`) is not refused creating a task.
- `internal/boardengine/taskguard_test.go` `TestTaskCreationGuard`: confirm a slug-free non-orch name such as `ly:webster` has a row that creates a task and passes.
- `plugins/ly/skills/board/SKILL.md`: confirm the new "Tasks from a run session" section states the refusal and the note way forward.
- `internal/boardcli/verbs_test.go:109`: confirm the doc comment of `stepTaskSessionFilesNotesOnly` says the note is accepted, not stored.
