# tender revive reads Claude Code placeholder after an API error as a typed draft and skips the session

`revive-reads-placeholder-draft` · Note · bug, tender, claude

## Seen

On 2026-11-26 two Claude sessions stalled on `API Error: Connection refused (ECONNREFUSED)`: `ly:rc-click-wrong-client-session:darn` and `ly:async-run-pusher:code-burler-fix`.
`lym tender revive` skipped both with reason `the input box holds a draft: "Resend your message to try again"`.
That text is Claude Code's own prompt after an API error, not text anyone typed, so revive never reaches the case it exists for.
No code on `origin/main` names the string.

## Workaround

The orch paused each run with `lym batten pause <slug>` and resumed it with `lym batten run <slug>`, which relaunches the row's agent on its own session.

## Expected

The input-box reader in `internal/shuttleengine/claudeengine` treats Claude Code's post-error prompt as an empty box, and `tender revive` gives the session its carry-on line.
