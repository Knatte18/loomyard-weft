# Find why a running run shows a dead waiter in the activity reader

`idle-notice-dead-waiter-cause` · Note · bug, batten, shuttle

## Seen (2026-10-24)

Task `idle-notice-no-live-run` landed as 450bc6ff1 with diagnostics only: `ReadAgentActivity` still skips a running run whose waiter pid is dead, so the idle notice can still say "no agent activity readable" while an agent works; it now logs that case (Warn once per episode). The root cause is unconfirmed.

## Idea

When the Warn line shows up in a run, read it with the run records and fix the cause: either the waiter pid recorded is wrong, or a live run legitimately has no waiter and the reader should read activity anyway.