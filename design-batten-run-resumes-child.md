# batten run brings its loom run back up as it was

`batten-run-resumes-child` · Note · enhancement, batten

`lyx batten run <slug>` on a run whose batten process had died, with its inner loom run halted (paused, blocked or failed), started batten, which logged "inner shed run halted; waiting for the operator to resume it" and waited without bringing the loom run back up; the orch had to run `lyx loom start --no-attach` in the task worktree as a second step.
Seen 2026-10-07 restarting the three bugfix runs after their batten processes died; the runs went on only once their drivers were also messaged.

Wanted (operator, 2026-10-07): `batten run` brings its current producer, the loom run, back up as it was, driver included.
The loom run itself stays where it halted, since that halt is its own; resuming it remains a separate step (a resume, or a message to its driver).
