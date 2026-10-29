# Each creel mail reaches the agent twice through the Stop hook

`creel-mail-arrives-twice` · Note · bug, creel, claude

## Seen (operator, 2026-10-29)

The operator sees a creel mail processed twice by the model: once when the hook fires, and once more when the model repeats what the hook delivered.

In the orch session each mail arrived as two messages: a `Stop hook feedback:` line carrying the mail, and a `Stop hook blocking error from command` message that repeats the mail after the whole hook command text.
Before each Stop-hook mail the session first gets `[creel] Mail is waiting for you`, ends its turn, and only then sees the mail.
The `PostToolUse` variant delivers the mail once, under a tool result.

## Idea

Find whether the Stop hook's exit code or output makes Claude Code surface the same text as both feedback and error, and make the hook deliver the mail once; the command text should not appear in the agent's context at all.
Check whether the hook-wake flake `TestScheduler_HookWake` is related.
