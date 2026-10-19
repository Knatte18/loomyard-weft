# A run started on one machine shows as running on another, and nothing stops the second machine from resuming it

`run-machine-ownership` · Note · bug, batten, board, orch, fabric

Imported from [issue #741](https://github.com/Knatte18/loomyard/issues/741).

Seen on a second Linux machine of the operator, in a fresh orch session of the loomyard hub.
The tasks hub-workspaces and implementer-seam were started on another PC.

On this machine:
- `lyx board list` showed them `running · Plan-Write` and `running · Merge-In`.
- `lyx batten status <slug>` showed `state: running` at `Run-Shed`, `error` empty, nothing since 2026-10-18 17:42 UTC.
- Neither pair worktree existed locally, and `lyx tender status` listed no session for either.

Nothing in the board, batten status or tender status says which machine owns a run.
The orch role reads a missing pair as a landed run, which is wrong here, because the board does not show either run as done.

`lyx fabric add <slug>` adopted both pairs from origin, and `lyx batten run <slug> --window` resumed both runs here without any check.
The driver lock is machine-local, so nothing stopped this machine from driving the same branches while the other PC may still have been driving them.

Workaround: the operator confirmed by hand that the other PC was not running them.

Expected:
- A run records the machine that drives it, and `lyx batten status`, the board and `lyx tender status` show it.
- A run owned by another machine is reported as such, not as a stalled or landed run.
- Resuming a run another machine owns is refused unless the operator hands it over explicitly.