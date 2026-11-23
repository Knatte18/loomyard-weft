# Resume an interrupted run agent's session instead of starting fresh

A run agent whose session ended before its work did, through a crash, a reboot or a killed pane, now resumes its recorded session when its row is called again.
Before, the re-call spawned a fresh agent: the files it wrote stayed in the worktree, but everything the session had learned was lost.
Only PR-Review, the sampler, the orch and account-switch relaunches resumed a session.

## What changes

`shuttleengine` resumes the newest earlier run of the same role when all of these hold:

- the new start declares the same output files as that run, names no session of its own and is awaited;
- the run's session ended unfinished: it died, lym stopped it as interrupted, or it reads running with its waiter gone and no relaunch holding it;
- the engine implements `SessionResumer` and its resume check accepts the session.

The resumed prompt opens with a preamble that tells the agent to re-read its task, check the disk and continue.
It takes no skill-load turn, and it carries no carried-over instructions, since the session already holds them.

A fresh agent still starts for a deliberate restart, and in these cases:

- a goto sets aside every interrupted run of the worktree (`loomcli/gotoresume.go`);
- a re-call after an escalation's answer, a new review round, and any run stopped on purpose, finished, timed out or ended on API errors;
- a session with no transcript on this machine, or one a live process holds: the resume check refuses and the start runs fresh.

`RunState` gains two optional fields:

- `Interrupted`, set when lym stops a run whose session had ended, through the step stop and the removal of a superseded strand;
- `NoResume`, set on a predecessor once a later start has taken its place, and on every interrupted run a goto sets aside.

`orch-role.md` now says that a re-call after a resumed halt whose agent ended early resumes its session.

## Design decisions

- Codex sessions keep starting fresh.
  The crash resume covers engines with a `SessionResumer`; no Codex resume path is built, because Codex is disabled in this hub and its subscription ends.
- A start whose resumed session never comes up sets both runs aside and retries as a fresh start, so a failed resume is not tried twice.
- A failed set-aside write is logged and never fails the start or the goto; the cost is that a later start may resume a session it should not.
- `carriedInstructions` now takes the predecessor `newestPredecessor` found, so one directory scan serves both the carry-over and the resume.

## To check by hand

- Code review left these open; check each:
  - Codex runs are excluded from the crash resume, in the `SessionResumer` check in `resumeInterrupted` (`internal/shuttleengine/crashresume.go`); this is the intended outcome of the decision above.
  - `removeSupersededStrands` in `internal/shuttleengine/attach.go` records a superseded running strand's stop as interrupted; check its test covers that path.
  - The start retry loop in `internal/shuttleengine/run.go` reuses the resume on a retried start; check a resumed start that fails to come up retries fresh.
- Kill a pane mid-row and re-call the row: the agent should resume its session and continue from the disk state.
- Run a goto after a crash: the rows it moves onto should start fresh sessions.

## Merge danger

**Door:** two-way
**Blast radius:** shuttleengine
