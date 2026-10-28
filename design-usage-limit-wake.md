# The watcher wakes the orch after a usage limit resets, and holds the repeat notices

`usage-limit-wake` · Note · feature, orch, batten, shuttle

## Seen

On 2026-10-28 a weekly usage limit stopped the Webster session of `validator-checks`, and batten sent the orch the same api-error notice again and again (#95 to #103, more after), until the limit reset at 14:00 Europe/Oslo.
The orch is hit by the same limit, so it cannot act on any of them; the notices only pile up.
After the reset the operator had to tell the orch, which ran `lym tender revive`.
Claude Code resumed some sessions on its own at the reset ("Usage limit reset · Continuing automatically"), others not.

## Idea (operator, 2026-10-28)

- When a session hits a usage limit, the orch watcher reads the reset time from the limit message itself, such as "resets 2pm (Europe/Oslo)"; `UsageLimitReader` in `internal/shuttleengine/hold.go` already tells a usage limit from an API error.
- While the limit stands, batten's usage-limit notices do not reach the orch; the watcher holds them, so no repeat notice is sent.
- One minute after the reset time the watcher wakes the orch with one line, such as "Usage limit reset at 14:00; run `lym tender revive` and check every run", and the orch revives and restarts what stopped, as it does on the operator's heads-up today.
- When no reset time can be read, the watcher retries waking the orch at a fixed interval, such as every 30 minutes, until the orch answers.
- `lym tender revive` stays harmless for a session Claude Code already resumed.
