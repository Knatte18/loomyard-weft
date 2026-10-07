# batten run resumes a halted inner run itself

`batten-run-resumes-child` · Note · enhancement, batten

`lyx batten run <slug>` on a run whose inner loom run is halted (paused, blocked or failed) starts batten, logs "inner shed run halted; waiting for the operator to resume it", and waits; the orch then has to run `lyx loom start --no-attach` in the task worktree as a second step.
Seen 2026-10-07 restarting the three bugfix runs after their batten processes died: each needed both commands, and batten sent a "child state paused" notice for each in between.

The wait is deliberate while batten is already running: a halt may be someone's intent (a pause for a goto), and batten must not override it.
But `batten run` is itself an explicit instruction to go on, so the operator wants it to resume a halted inner run itself (operator, 2026-10-07).
Only a halt met while batten is already running waits for a resume.
