# Each creel mail reaches the agent twice through the Stop hook

`creel-mail-arrives-twice` · Note · bug, creel, claude

## Seen (operator, 2026-10-29)

The operator sees a creel mail processed twice by the model.
The terminal shows `Ran 2 stop hooks`, then the mail once as `Stop says:` and again as `Stop hook error:`, both with the same `[creel] #<id> from ...` text.

In the orch session each mail reaches the model as two messages: a `Stop hook feedback:` line carrying the mail, and a `Stop hook blocking error from command` message that repeats the mail after the whole hook command text.
The `PostToolUse` variant delivers the mail once, under a tool result.

## Found

The shuttle run's `settings.json` registers two Stop hooks:

- a command hook that runs `lym creel hook-read claude` and delivers the mail;
- a second hook that runs `lym creel hook-wait claude` with `asyncRewake: true`, the ring that wakes an idle strand with the bell line `[creel] Mail is waiting for you`.

Which of the two produces the second copy is not yet known.

## Idea

Find whether the Stop hook's exit code or output makes Claude Code surface the same text as both feedback and error, or whether both hooks deliver it, and make the mail reach the agent once; the hook command text should not appear in the agent's context at all.
Check whether the hook-wake flake `TestScheduler_HookWake` is related, and the note `creel-ring-carries-mail`.
