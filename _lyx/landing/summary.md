# Refuse board writes that make a task from a run session
A run's agent could file a follow-up as a task with `lyx board upsert`.
When that task depended on the run's own entry, the Land notice named it "newly ready" and the orch started it as a full run nobody approved.
Only the operator decides what becomes a task, with which recipe and profile.
Now a board write from a run agent's session is refused when it leaves a task whose slug was not a task before, and the agent is told to file a note instead.

## What changes

- `internal/boardengine`: the write critical section runs a new net-change check, `checkTaskCreation` in `taskguard.go`, beside the run-lock check.
  It covers a new task, a promote, a note upserted to kind task and a merge that upserts a new task.
  A new note, an edit of an existing task and a rename pass.
  The refusal names the strand and the entry, and says to file the follow-up as a note with `"kind":"note"`, which the operator may promote.
- `internal/boardengine.Config` gains `StrandName`.
  `internal/boardcli` sets it from `LYX_STRAND_NAME` for every write verb, so `upsert`, `upsert-batch`, `merge` and `promote` are all covered.
- The Discussion, Plan and Darn writer stencils say a follow-up becomes a note, never a task, and never promotes one.
  The discussion and plan review rubrics flag a decision or card that files or promotes a task.
- `plugins/ly/skills/board/SKILL.md`, and the `doc.go` of both packages, document the refusal.

## Design decisions

- One net-change check on the loaded versus the saved entries covers every write path, instead of one guard per verb.
  A rename is judged with the loaded slug renamed, so it creates no task.
- A run agent is a strand name that parses with a task slug under any role but the orch role, numbered orch roles included.
  An empty or slug-free name (the operator at a terminal, the hub orch) and an orch thread in a task worktree write as before, since those act for the operator.
  This follows how the board CLI's `run_settings` guard already judges a task session.
- The strand name is read in `boardcli` and passed in, never read in `boardengine`.
- The guard reads only the name the caller passes, so it stops a mistaken write by a run's agent, not one that clears its environment.
- A merge from a run agent whose upserted slug is a new task is refused even when it folds existing tasks, since curating tasks is the operator's call.
- The orch role stencil is unchanged: it keeps starting each task named after `newly ready:` at once.
  With this guard only the operator creates tasks, so every task on the board is already approved.

## Not covered

A Go producer that writes the board in-process is not guarded, since only the CLI sets the strand name.
No producer creates a task today.

## To check by hand

The new tests in `internal/boardengine/taskguard_test.go` and `internal/boardcli/verbs_test.go` cover the refused and passing cases.
Not run by hand: a real run session attempting `lyx board upsert` of a new task, and the stencil wording in a live Discussion or Plan.
