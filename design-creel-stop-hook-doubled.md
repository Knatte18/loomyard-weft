# Stop-hook mail still arrives as feedback plus a blocking error that carries the hook command

`creel-stop-hook-doubled` · Note · bug, creel

## Seen

After the landing of `creel-hook-delivery-fixes`, the orch session still gets every creel mail as a pair of Stop-hook messages (operator, 2026-10-29).
The mail text itself reaches the model once now, in the hook's additional context, but the surrounding messages are doubled and carry the hook command.

One mail arrives in two steps:

1. The `hook-wait claude` ring wakes the idle strand with a system notification titled `Stop hook feedback`, whose body is a `Stop hook blocking error from command "Stop"` holding the bell line `[creel] Mail is waiting for you. End this turn now; ...`.
2. When the turn ends, the `hook-read claude` hook delivers the mail as a user turn that holds both a `Stop hook feedback:` line, `[creel] Mail for you arrived as this turn ended; ...`, and a `Stop hook blocking error from command:` message that repeats that same line after the whole shell text of the hook command, with the mail in `Stop hook additional context`.

So the model reads the pointer line twice in step 2, and the full hook command text, with its paths and flag-file logic, sits in the agent's context.
The earlier task's fix text said the hook command text should not appear in the agent's context at all; it still does.

## Fix

- Find why the harness shows the `hook-read` result both as feedback and as a blocking error, and whether the exit code, the `decision` field or the `reason` field of the hook's JSON causes the second copy.
- Make the pointer line reach the model once and keep the command text out of the context.
- Decide whether the step 1 bell and the step 2 delivery can be one message; this is the point of `creel-ring-carries-mail`, so fold the two notes together if the cause is shared.
