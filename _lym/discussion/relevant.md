# Relevant

- `internal/slackengine/inbound.go` `route`, `mentionedSlug`, `SetPresence`, `inbound.poll`, `inbound.handle`, `inbound.forward`, `inbound.cursorFor` — the poll and `@<slug>` routing that push and per-channel routing replace
- `internal/slackengine/outbound.go` `outboundText`, `programSlug` — sender rendering and answer hint that D8 changes, and the program slug parse it reuses
- `internal/slackengine/doc.go` — package documentation describing the relay that D10 rewrites
- `pattern/PATTERN-agent-addressing.md` — the Slack boundary rule the tokens change
- `pattern/PATTERN-daemon-wakeups.md` — lists the relay's loops that the poll's removal changes
- `pattern/PATTERN-child-processes.md` — the relay's double fork and hub-owner record, kept as is
- `pattern/PATTERN-lym-files.md` — placement of the hub's new files and the machine lock
