# Dobby never starts a task whose slug has an old finished run record, and status gives no reason

`dobby-start-blocked-silently` · Note · bug, dobby, batten

## Seen (operator and hub orch, 2026-10-29)

The released task `claude-md-lym-rules-out` did not start for over an hour.
`lym beam status` showed it as `ready to start`, with room under the run limit and no dependency, and gave no reason.
`.lym/beam/dobby.log` showed dobby trying to start it about every 30 seconds from 13:43:17Z, each pass failing with `battencli: "claude-md-lym-rules-out" has already completed; to run it again, run "lym batten cleanup --apply" from the prime ...`.
The cause was an old run record for the same slug, `_lym/shed/claude-md-lym-rules-out/`, from machine `neve`, 2026-10-28, ended at Worktree-Teardown.
This hub's prime is on `osl-1046`, and the operator knows of no run of that slug on Neve; why the record read as done is not known.
`lym batten cleanup --apply` removed the record, after which no branch of that name was left locally or on the remote.

## Idea

- `lym beam status` states why a ready task is not started, with the refusal dobby last got, so `ready to start` is never shown with no reason.
- The start of a task whose slug has an old finished run record is handled where the task is released or where dobby starts it, with a message that names the way forward, instead of failing every pass.
- Find out where a run record of another machine came from for a slug this hub had not run.
