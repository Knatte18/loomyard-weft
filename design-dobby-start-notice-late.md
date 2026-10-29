# Dobby's notice for a start that keeps failing reached the orch over an hour late

`dobby-start-notice-late` · Note · bug, dobby, creel

## Seen (operator and hub orch, 2026-10-29)

Dobby failed to start the released task `claude-md-lym-rules-out` on every pass from 13:43:17Z, about every 30 seconds, and the orch heard nothing.
Its mail, creel #34 from `ly:DOBBY` with the text `dobby could not start the task`, reached the orch only after more than an hour, right after the orch ran `lym batten cleanup --apply`.
The orch found the cause by asking `lym beam status` and reading `.lym/beam/dobby.log` through a fork, after the operator noticed the task had not started.
The role says dobby sends a line for a refused start and for a pass that keeps failing, so the notice should have come within a few passes.

## Idea

- Find out why the notice came late: whether dobby sends it only after many failed passes, whether the send was held, or whether creel delivered it late.
- A refused start or a pass that keeps failing reaches the orch within a few passes, and a start that keeps failing with the same error does not resend it every pass.
