# Unreachable origin halts the run as failed repeatedly instead of one retryable stop

`unreachable-origin-halts-failed` · Note · bug, shed, landing, fabric

Imported from [issue #243](https://github.com/Knatte18/loomyard/issues/243).

An unreachable origin halted the run as `failed` with anomaly `producer-hard-failure` seven times in a row, across PR-Review and Finalize, instead of waiting or blocking once.

What happened:
- PR-Review and Finalize halted with `landingshed: ... push the task branch ... fabricengine: ls-remote of <task>-weft ... Recv failure: Connection reset by peer`.
- Finalize then halted three more times with `landingshed: Finalize: read who owns the run: board: origin is unreachable, so the owner write was not pushed: fetching origin for the board ... failed (gitrepo: ... git exited 128 ...)`.
- One halt in between was `blocked` with anomaly `escalation-to-human`: `owner check: origin was unreachable, so who owns this run could not be judged; re-run "lym loom resume" in the task worktree once origin answers`.
- Each halt wrote a note and a history entry; the history shows `Finalize / stuck` then `Finalize / goto` before each re-failure.

Why it is abnormal:
- A transient network failure is classed as a hard failure, the same class as a producer defect, and the run state becomes `failed`.
- The same condition surfaced under three different halt shapes (`failed`, `blocked`, `failed`), so the operator cannot tell one outage from three problems.
- Each re-step re-ran immediately and failed again within about a minute, so the retries consumed attempts without waiting for the remote.

What to do:
- Classify unreachable-origin errors (ls-remote, fetch exit 128, connection reset) as one retryable or `blocked` kind with a single note, and have the driver wait or back off before re-stepping.
- Keep `failed` for errors that retrying cannot fix.

Provenance: task `retire-d9`; notes loom-halt-3.md, loom-halt-4.md, loom-halt-5.md, loom-halt-6.md, loom-halt-7.md, loom-halt-8.md, loom-halt-9.md.
