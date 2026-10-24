# Decision record — runs-never-write-board

## Goal

A run never files a task: only the operator decides what becomes a task.
A run may at most file a follow-up as a note, and the orch starts a newly ready task only when the operator approved its start.

## Scope

- `internal/boardengine`: a board write refused when its net change leaves a task that was not a task before, for a board configured with a task session's strand name.
- `internal/boardcli`: every write verb's board carries the calling strand name, so `upsert`, `upsert-batch`, `merge` and `promote` from a task session cannot create or promote a task.
- Stencils: Discussion, Plan and Darn writers, and the discussion and plan review rubrics, say a follow-up becomes a note, never a task.
- The orch role stencil: a task named after `newly ready:` starts only when the operator approved its start.
- Docs: `internal/boardengine/doc.go` and `internal/boardcli/doc.go`.

## Decisions

### D1: The refusal lives in the board's write critical section

The write critical section already compares the loaded entries with the entries after the mutation for the run lock.
A second check there refuses any entry that is a task after the write while no loaded entry of its slug was a task, which covers a new task, a promote, an upsert of a note to kind task and a merge whose upserted entry is a new task, through every verb.
A rename is judged with the loaded slug renamed, so it creates no task.
Rationale: one net-change check covers every write path, the way the run lock does, instead of one guard per verb.

### D2: A task session is named by the strand name, and the orch role is exempt

`boardengine.Config` gains a field holding the calling session's strand name, which boardcli sets from `LYX_STRAND_NAME`.
The engine refuses task creation when that name parses as a name carrying a task slug and its role is not the orch role.
An empty or slug-free name, the operator at a terminal and the hub orch, writes as before, and so does an orch thread in a task worktree, which is the operator's own channel.
Bound: the guard reads the strand name the caller passes and nothing else, so it stops a mistaken write from a run's agent, not a determined one that clears its environment;
it never touches a note write, a status write, a run_settings write or the replay of a local commit.
Rationale: the board CLI's run_settings guard and the hub-wide config guard already judge a task session by the slug in its strand name.

### D3: The refusal names the way forward

The message names the strand and the entry, and says to file the follow-up as a note with `"kind":"note"`, which the operator may later promote.
Rationale: the agent that hits it needs the allowed alternative in the same message.

### D4: Stencils say a follow-up becomes a note

The Discussion, Plan and Darn writer stencils, beside their existing own-entry rule, say a decision, card or step that files a follow-up files it as a note, never a task, and never promotes one.
The discussion and plan review rubrics flag a decision or card that files or promotes a task.
Rationale: the writer learns the rule before designing a step the board would refuse.

### D5: The orch starts a newly ready task only with the operator's approval

The orch role stencil's landed-notice rule starts each task named after `newly ready:` only when the operator approved its start, and otherwise proposes it to the operator.
Rationale: a task an operator approved together with its start still starts at once, while a task nobody released waits.

### Added after Discussion (parent, 2026-10-24): Drop D5: the orch keeps starting newly ready tasks at once

Decision: D5 is withdrawn: the orch role stencil's landed-notice rule stays as it is, starting each task named after `newly ready:` at once. Only the operator creates tasks once this run's board guard lands, so every task on the board is operator-approved and needs no second approval to start.

Rationale: Operator, 2026-10-24: tasks the operator approved start automatically; the fault was a run filing a task, which D1-D4 close.

## Constraints

- PATTERN-cliwire-sole-wiring and the existing boardcli wiring: the strand name is read in boardcli, never in boardengine.
- PATTERN-tolerant-records: no record format changes.
- Stencil text follows semantic line breaks.

## Auto-mode assumptions

- An orch thread in a task worktree acts for the operator and may create tasks.
- A merge from a task session whose upserted slug is a new task is refused even when it folds existing tasks, since curating tasks is the operator's call.

## Open risks

- A Go producer that writes the board in-process is not covered, since only the CLI sets the strand name; no producer creates a task today.

## Acceptance criteria

- From a task session's strand name, an upsert of a new task, an upsert turning a note into a task, a promote, an upsert-batch carrying a new task and a merge upserting a new task are refused, and board.json is unchanged.
- From a task session, a new note, an edit of an existing task that is not run-held, and a rename pass.
- From an empty strand name, the hub orch's slug-free name and an orch thread's `<shortname>:<slug>:orch` name, creating a task passes.
- The refusal message names the note way forward.
- The boardcli verbs pass the strand name through, shown by an `upsert` of a new task refused under a task strand name.

## Notes for the plan writer

- `checkRunLocks` in `internal/boardengine/runlock.go` is the model for the net-change check.
