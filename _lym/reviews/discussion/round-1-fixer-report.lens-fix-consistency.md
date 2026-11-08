```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling decisions, criteria and cross-references they touch, and find any twin text left unchanged.

**MEDIUM: D2's claim that the relay's own writes never wake it contradicts D2's hint removal and D7's presence writes**
- Changed text: the D2 Rationale sentence "The watch sees no file the relay writes itself, so its own records and log never wake it", and the new D2 acceptance bullet "the relay's own writes to its Slack directory wake nothing".
- Summary: the relay removes hints from the watched inbox directory, and it writes the presence file the watch takes. Both raise file events, so the relay's own writes do wake it.
- Evidence:
  - D2 says the relay "removes any other hint unread", and draining the inbox consumes hints.
  - `internal/fswatch/fswatch.go:108-119` (`operationName`) delivers `remove`, `create` and `write` alike.
  - D7 has the relay's `away` and `back` set the hub's presence, which is `presence.json`, the file D2's names filter watches.
- The one-second floor bounds the loop. Even so, the rationale sentence and the acceptance bullet state a behaviour the design does not have.

**LOW: D3's creator test has no stated source of the creator on the D6 reopen and D8 `not_in_channel` paths**
- Changed text: D3, "Each call is made only for a channel … whose creator is this machine's bot user (D5)".
- Summary: only D5 step 2 reads the creator, through `conversations.list`. D6's reopen of a map-held archived entry and D8's `not_in_channel` invite name no read of it, so those two user-token paths cannot apply the test as written.
- Evidence: D6 says "To reopen, the relay calls `conversations.unarchive`…" with no lookup. The D8 `not_in_channel` sentence says only "under D3's bound".

**LOW: a D3 acceptance bullet still says "the relay start before any spawn", against D3's new wording**
- Changed text: D3 now reads "The relay start, `lym slack start` and `lym slack away` refuse through it". A new bullet says "a relay started with only a bot token mails … and exits".
- Summary: the twin bullet "a machine missing any token is refused by the relay start before any spawn and by `RequireConfigured`" was kept.
- Evidence:
  - The relay start (`RunRelay`, `internal/slackengine/relay.go:186`) runs after the spawn.
  - Only the `start` and `away` verbs refuse before a spawn (`internal/slackcli/cli.go:358`, `:430`).
  - Under D3, `EnsureRelay` answers quietly and spawns nothing.
- The bullet should name `lym slack start` and `lym slack away` in place of "the relay start before any spawn".

Outside my lens: login confirming an `xapp-` token by opening a Socket Mode connection gives the app a second connection while a holder runs. Slack may hand that login connection events it never acknowledges, which the fix-proposal and fix-claims lenses may want to weigh.

No further tool calls were made.
