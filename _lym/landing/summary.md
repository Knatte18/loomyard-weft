# The orch ends its own earlier process and resumes its session

An orch start used to fall back to a fresh launch on any failed resume.
On both recorded adopts the orch's earlier Claude process was still alive after its strand was removed and held the transcript, so the resume refused and the start launched a second orch with only the carry-over.
Now the orch registers its Claude process, a start ends its own earlier holder and resumes the same session, and a fresh launch happens only when this machine has no transcript.

## What changes

- `runproc` gains `RoleOrch`, `RegisterOrch` and `IsRegistered`.
  `orchcli` registers the started orch's provider process under the worktree's owner key.
  The record outlives the strand's removal and is pruned when the process dies.
- `launchOrch` no longer falls back to a fresh launch on any resume error.
  When `CheckResume` refuses for a live holder:
  - a holder registered as this worktree's orch is ended, and the start resumes the same session id once more;
  - any other holder refuses, naming its pid and the way forward, and nothing launches fresh.
    An orch the pre-upgrade `lym` launched is unregistered, so the first start after the upgrade refuses over it once.
- A fresh launch with the carry-over happens only when the refusal is `shuttleengine.ErrNoTranscript`.
  The start says so in its warnings, naming the abandoned session and the number of carried instructions.
  The session id stays in the machine-local orch state, and no tracked record names it.
- `claudeengine` returns a `*shuttleengine.SessionHeldError` naming the holder's pid, and wraps `ErrNoTranscript` for a missing transcript.
  It implements the new optional `shuttleengine.SessionHolderFinder`, which `Run.SessionHolder` exposes so `orchcli` can learn the pid to register.
- `runproc.EndSubject` and `pairteardown.EndSubjectProcesses` end every process this machine runs for a subject another machine now owns:
  each run's processes, then the subject's own, then the dobby when the caller asks.
  Roles that serve the whole hub (adopt, sync, push, test-map, Slack relay) and teardown children are left alone.
- The orch watcher's owner check (`orchengine.OwnerWatch.End`) calls it without the dobby, which hands the subject over itself.
  The strand is stopped even when the end fails, and the exit is recorded either way; a failed end is logged at Warn.
- `pairteardown` spares `RoleOrch` when it ends a pair's processes, since only a subject another machine owns ends its orch.
  `fabriccli`'s anchor move ends the orch role along with the dobby's push and sync roles.

## Design decisions

- `EndSubjectProcesses` has no caller for the dobby pass here.
  The dobby's handover and the catch-up in `bolt-holds-catch-up` are the other callers the function is exported for.
- A holder is registered only by `lym`.
  An unregistered live holder is never ended, since it may be an operator's own session.
- `claudeengine` keeps the registry layout and the holder rule in one place (`liveHolder`), shared by the resume check and the holder read.

## To check by hand

- Summary and doc text of the touched packages may still describe the earlier shape of this change rather than the code as it stands.
- Start an orch, remove its strand while its Claude process stays alive, and run `lym beam start` again.
  It should end that process and resume the same session id.
- Start with a live Claude process on the orch's session that `lym` did not register.
  The start should refuse, naming its pid.
- Start on a machine with no transcript of the recorded session.
  It should launch fresh, carry over the instructions, and say so.
