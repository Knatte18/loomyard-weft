```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every claim the fix added or rewrote against the code at HEAD.

**Claims that hold:**
- `LastSlackOK` is what the last-healthy time is today. `recordingSlack` stamps it only after a Web API call succeeds (`internal/slackengine/relay.go:117-133`).
- The restart cap reads it. `lifetime` compares it against `relayHealthyLifetime`, which is 10 minutes (`start.go:43`, `:215`), and only on the while-away path (`start.go:202`). So the "ten minutes" criterion and the "`EnsureRelayWhileAway`'s restart cap" Notes bullet are both right.
- `lym slack status` reads the same field (`internal/slackcli/cli.go:533-534`).
- An outbox record exists (`geometry.go:21-22`, `outbound.go:39-43`). The rewritten D2 criterion names real records.
- `hubregistry.Live` fails when `readEntries` fails (`hubregistry.go:98-102`, `:119-125`).
- The refused-read notice that the new constraint line rewords does name `lym slack setup` (`inbound.go:297`).
- `botScopes` lists the old scope set (`cli.go:31`).

**Findings:**

- **LOW.** D3's new sentence: "Its way forward is moving that record aside, after which each hub records itself again at its next `lym creel inbox`."
  - Summary: a hub does not always record itself at its next inbox. The daemon records at most once per `hubRecordInterval` after a record that succeeded.
  - Evidence: `internal/creelengine/hubs.go:66-82` says "After a record that succeeded, it records at most once per hubRecordInterval". The existing refusal words it as "lym creel inbox in each hub, a minute on, records it again" (`internal/hubregistry/hubregistry.go:148`).

- **LOW.** D3's rewritten sentence: "It then mails the hub's orch one `[slack relay]` notice … posts the line below, and exits."
  - Summary: it says the relay always posts, but the two sentences after it post only when there is a bot token and either a map entry or an old setup channel key. With neither, nothing is posted.
  - Evidence: the following sentences open "When it has the bot token and the map holds the prime's channel…" and "When the map does not hold the prime's channel and `setup.json` still carries an older binary's channel key…".

Tool calls:
1. `sed`/`grep` of `start.go:195-222`, `hubregistry.readEntries` and creelcli's record callers. My context lacked the restart cap's call site and how the hubs record is read and written.
2. `sed`/`grep` of `creelcli/hubdirectory.go:40-60`, `hubregistry.refuseUnreadable` and creelengine's `Record` caller. My context lacked the existing way forward for an unreadable hubs record.
3. `sed` of `creelengine/hubs.go:66-92`. My context lacked when the daemon records its hub.
