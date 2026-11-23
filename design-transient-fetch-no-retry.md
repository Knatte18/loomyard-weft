# Runs stop and escalate on a transient GitHub fetch failure instead of retrying

`transient-fetch-no-retry` · Task · bug, git, batten, landing · running · Drift-Baseline

Imported from [issue #193](https://github.com/Knatte18/loomyard/issues/193).

## Seen

Around 10:08 and again at 10:33 on 2026-11-23, GitHub was unreachable from this host for a short time.
`git fetch` failed with `HTTP/2 stream 1 was closed cleanly, but before getting all response header fields`, and `git ls-remote` failed with HTTP/2 stream resets and HTTP/1.1 connection resets.

Three batten runs in one hub stopped on it within a minute of each other and escalated to the parent orch as questions:

- `friction-profile-rules` failed at Run-Shed: `battencli: origin reconcile ... git fetch origin +refs/heads/<parent>:refs/remotes/origin/<parent>: exit 128`.
- `models-wemod-robustness-sanity` blocked at Finalize with a landing stop: the PR had been merged on GitHub, but the parent's upstream could not be fetched.
- `models-torque-drag-adapter` stopped at the start of its run on `git ls-remote origin`.

Each stop's tackler or arbiter handed the question back ("should the run be re-launched once GitHub answers?"), two of them saying no parent was recorded for the run.
A minute later `git ls-remote origin` answered at once.

## Workaround

The orch checked `git ls-remote origin`, then ran `lym batten run <slug>` for the two batten stops and `lym loom resume` in the task worktree for the Finalize stop.
All three went on without any other change.

## Expected

A transient network failure on a git fetch, push or ls-remote is retried by lym with backoff, for a bounded time, before the run stops.
Only a failure that outlasts the retries reaches the parent, and then as a notice naming the failing remote operation, not as a question.


## From issue #203

Imported from [issue #203](https://github.com/Knatte18/loomyard/issues/203).

Seen: on a machine with an unstable connection, one git ls-remote origin failure (HTTP/2 stream closed before response headers, exit 128) halted a darn run at Run-Shed's origin reconcile. The tackler retried twice within seconds, handed the stop back, and the parent orch got a question. A hand pass right after was refused with 'origin is unreachable', and a third pass a minute later went through. Weft pushes of the board and of a records branch failed the same way, each with its own notice. 10 of 10 ls-remote calls succeeded a few minutes later.

Workaround: the orch re-ran lym batten run by hand until origin answered.

Expected: network calls to origin retry with backoff over a minute or more before a run halts, and a stop whose cause is a transient network error is retried by dobby rather than handed to the parent as a question.