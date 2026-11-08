```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims, checking each claim the fix added or rewrote against the code at HEAD.

**MEDIUM. D2 Rationale "The watch sees no file the relay writes itself, so its own records and log never wake it", and the D2 acceptance bullet "the relay's own writes to its Slack directory wake nothing".**
Summary: the relay does write files the watch sees, so this claim and the bullet built on it are false.
Evidence:
- The relay writes the presence file when an operator sends `away` or `back`. `internal/slackengine/inbound.go:329` calls `SetPresence`, which calls `WritePresence(geom.PresencePath…)` at `inbound.go:172`. D7 keeps that path, and the presence file is the one name the new watch takes.
- The relay removes the hints it drains, and D2 says it "removes any other hint unread". `fswatch` delivers a `remove` op for any entry of the watched inbox directory (`internal/fswatch/fswatch.go:116`, with no names filter on the inbox).
- A holder relay also writes `resync` and channel hints into its own hub's inbox (D1, D2).
- The one-second floor bounds each of these to one extra wake, but the stated guarantee does not hold. A test written to the acceptance bullet would fail on a presence change made from Slack.

**LOW. D3 "When it has the bot token, it posts the same line to the prime's channel the map holds", and the D3 acceptance bullet "posts the same line to the prime's channel, and exits".**
Summary: the bullet asserts the post with no condition, but the Decision makes it conditional on the map holding the prime's channel.
Evidence: in the upgrade case F11 describes, no `channels.json` exists yet. D5 ensures the prime's channel at relay start, which comes after the token check that exits. So the map holds no prime channel and nothing can be posted.

**LOW. Constraints "The Socket Mode `hello` message carries `connection_info.app_id` (\"Using Socket Mode\")".**
Summary: this is stated as a documented fact, but the docs only show it in an example. D3's app-level match rests on it.
Evidence: the web-fetch report says "The page does not describe `connection_info` in prose. It appears only in this example."

These claims hold:
- `RequireConfigured` is called by `start` (`internal/slackcli/cli.go:358`), `away` (`cli.go:430`) and the relay (`internal/slackcli/wiring.go:188`, `internal/slackengine/relay.go:186`).
- `ensure` answers quietly (`internal/slackengine/start.go:180-181`).
- Bring-up warns (`internal/orchcli/start.go:549`) and the anchor change returns the error (`internal/fabriccli/anchordeps.go:282`).
- `hubregistry`'s way-forward text matches `internal/hubregistry/hubregistry.go:93`.
- `refusalCap` is 5 (`internal/slackengine/outbound.go:32`).
- The stale count is hub-wide (`outbound.go:46`).
- `FromHostname` cuts at the first dot (`internal/machinename/machinename.go:23`).
- `hubregistry.Live` exists (`internal/hubregistry/hubregistry.go:98`).
- `fswatch.Watch` has a names filter (`internal/fswatch/fswatch.go:33-34`).

Tool calls:
1. grep of `machinename.FromHostname`, `hubregistry.Live` and import-rule tests: my context lacked those files, to check the D4 and D1/`PATTERN-lym-files` claims.
2. grep of `SetPresence`/`WritePresence` in `internal/slackengine`: my context lacked whether the relay itself writes the presence file, to check the "sees no file the relay writes" claim.
