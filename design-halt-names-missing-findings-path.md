# loom-halt note and blocked reason name a gate-findings path that does not exist

`halt-names-missing-findings-path` · Note · bug, loom, gate

Imported from [issue #174](https://github.com/Knatte18/loomyard/issues/174).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `.lym/shed/`, `.lym/shuttle/`.

**What happened.**
The halt reason for a Darn-Write gate failure says "findings:" followed by a path under `.lym/shed/<slug>/gate-findings/Darn-Write.md`.
That file does not exist; the `.lym/shed/<slug>` directory is empty.
The trace log names a different path, `.lym/shuttle/<run>/gate-findings.md`, and that run directory is gone too.

**Why abnormal.**
The halt note and the `lym loom resume` way forward point the operator at findings that cannot be read.
The cause of the block is only recoverable from the trace log.

**What to do.**
Copy the findings file to the stable path the halt reason names before the shuttle run directory is removed, or name the path that survives.

Provenance: task raddle-slug-length-check; note loom-halt.md.
