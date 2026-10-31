# Dobby says why a task is not started, and tells the orch of a failing start in time

`dobby-start-failures` · Task · bug, dobby, batten, creel · running · Describe

## Seen (operator and hub orch, 2026-10-29)

The released task `claude-md-lym-rules-out` did not start for over an hour.
`lym beam status` showed it as `ready to start`, with room under the run limit and no dependency, and gave no reason.
`.lym/beam/dobby.log` showed dobby trying to start it about every 30 seconds from 13:43:17Z, each pass failing with `battencli: "claude-md-lym-rules-out" has already completed; to run it again, run "lym batten cleanup --apply" from the prime ...`.
The cause was an old run record for the same slug, `_lym/shed/claude-md-lym-rules-out/`, from machine `neve`, 2026-10-28, ended at Worktree-Teardown.
This hub's prime is on `osl-1046`, and the operator knows of no run of that slug on Neve; why the record read as done is not known.
`lym batten cleanup --apply` removed the record, after which no branch of that name was left locally or on the remote.

The orch heard nothing of the failing start: its mail, creel #34 from `ly:DOBBY` with the text `dobby could not start the task`, reached it only after more than an hour, right after it ran `lym batten cleanup --apply`.
The orch found the cause by asking `lym beam status` and reading `.lym/beam/dobby.log` through a fork, after the operator noticed the task had not started.
The role says dobby sends a line for a refused start and for a pass that keeps failing, so the notice should have come within a few passes.

## Design

- `lym beam status` states why a ready task is not started, with the refusal dobby last got, so `ready to start` is never shown with no reason.
- The start of a task whose slug has an old finished run record is handled where the task is released or where dobby starts it, with a message that names the way forward, instead of failing every pass.
- A refused start or a pass that keeps failing reaches the orch within a few passes, and a start that keeps failing with the same error does not resend it every pass.
- Find out why the notice came late: whether dobby sends it only after many failed passes, whether the send was held, or whether creel delivered it late.
- Find out where a run record of another machine came from for a slug this hub had not run.
