# A run's agent files follow-ups as notes, never as tasks

Only the operator decides what becomes a task.
A run's agent could still create one with `lyx board upsert`, and a task that depended on the running task was then reported "newly ready" at landing and started as a full run nobody approved.
The board now refuses that write, and the writer stencils tell agents to file a follow-up as a note.

## The board guard

`boardengine.Config` gains `StrandName`, the agent name of the writing session, and boardcli sets it from `LYX_STRAND_NAME` on every verb.
The write critical section, beside the run lock, runs `checkTaskCreation` on the net change from the loaded entries to the entries after the mutation.
For a run agent's name, it refuses any entry that is a task afterwards while no loaded entry of its slug was a task.
That covers a new task, `promote`, an upsert turning a note into a task, an `upsert-batch` item and a `merge` upserting a new task.
A rename is judged with the loaded slug renamed, so it passes, as do notes, status writes and edits of existing tasks.
The refusal names the session and the entry and gives the way forward: file the follow-up with `"kind":"note"`.
Nothing reaches disk on a refusal.

A run agent's name is one carrying a task slug under any role but `orch`.
An empty name (the operator at a terminal), the hub orch's slug-free name and an orch thread's `<shortname>:<slug>:orch` name create tasks as before.
The guard trusts the name its caller passes, the same bound as the existing run_settings and hub-wide config guards: it stops a mistaken write, not a determined one.
The replay of a local board commit never runs it.

## Stencils

The Discussion, Plan and Darn writer stencils now say a follow-up goes on the board as a note and no decision, card or step promotes a note.
The discussion and plan review rubrics gain a bullet that flags a Decision or card that files or promotes a task.

## Design decisions

- The check lives in the engine's write critical section rather than in each CLI verb, so one net-change comparison covers every write path, the way the run lock does.
- An orch thread in a task worktree is exempt because it is the operator's own channel. This departs from the run_settings guard, which refuses every slug-bearing strand.
- A merge from a run agent whose upserted slug is a new task is refused even when it folds existing tasks: curating tasks is the operator's call.
- The orch role stencil is unchanged. The parent's recorded decision dropped D5, so the orch still starts newly ready tasks at once, since every task is now operator-made.

## Tests

`TestTaskCreationGuard` (tier0) pins each refused write and each allowed one, including the exempt names, and checks that board.json stays byte-identical on a refusal.
The boardcli `TestCLI` step `TaskSessionFilesNotesOnly` shows `upsert` passes the strand name through: under a task strand, a new task is refused and the same entry as a note is stored.
