---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/slackcli/cli.go:303
    summary: endLiveRelays returns at the first hub whose stop fails, so later hubs keep a relay on the replaced tokens and are never named
    fix: In endLiveRelays, do not return inside the loop. Collect each hub whose stop failed, with its error, and keep ending the remaining hubs. After the loop, fill relays_ended with the hubs ended, then return one error that names every hub not ended with its error and says both tokens are stored and the way forward is `lym slack stop` then `lym slack start` in each of those hubs. Add one row to the Login relays-ended test, with the first of two hubs failing to stop, that expects the second hub ended and both hubs named.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/slackcli/cli.go:269
    summary: the app match compares helloAppID with botAppID and passes when both are empty, so two tokens of unproven app are stored
    fix: In login, before the helloAppID != botAppID comparison, refuse when either id is empty. Use the same way forward as the mismatch refusal, naming the token kind whose app id Slack did not give and saying nothing was stored. Add a row to the app-id refusal test in cli_test.go for an empty bots.info app id and an empty hello app id, each expecting the refusal and no write.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/slackcli/wiring.go:133
    summary: liveHubGeometries, the production live-hub seam, has no test, so its error wording and the geometry it builds per hub are unpinned
    fix: Add a test in internal/slackcli that drives liveHubGeometries against a hub registry record written in a temp user-state directory. Make the directory injectable, or set the environment variable standalonestate.UserStateDir reads in a non-parallel test that says why it is not parallel. One row expects one geometry per live hub, with HubPath set. A second row has an unreadable record and expects an error that names the record's path and wraps the seam's error. A third row has a hub whose geometry fails and expects an error that names that hub's path.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/slackengine/socket.go:32
    summary: the Ack comment says Slack asks for an acknowledgement within three seconds, which the decision record says Slack does not state
    fix: In the doc comment of Socket.Ack, replace "which Slack asks for within three seconds of delivering it" with a statement that an unacknowledged envelope is sent again by Slack, with no deadline named.
---

### [LOW] endLiveRelays stops at the first failed stop

**Location:** internal/slackcli/cli.go:303
**Issue:** Login has stored both tokens before it ends the live hubs' relays. The loop over `hubs` returns at the first `c.stop` error. Every later hub keeps a relay running on the replaced tokens, and the error names only the failing hub. D3 says a login that changes a stored token ends the relay of each live hub and names each one. The way forward in the error, `lym slack stop` then `lym slack start` in that hub, leaves the unnamed hubs on the old tokens.

### [LOW] An empty app id on both sides passes the app match

**Location:** internal/slackcli/cli.go:269
**Issue:** D3 writes both tokens only when the app-level token's app id is the bot token's. The check is `helloAppID != botAppID` with no test for empty. If `bots.info` and the `hello` message both give an empty `app_id`, the two tokens are stored as one app's. The record notes that no Slack prose describes the `hello` field, so an absent value is plausible, and the stored `AppID` would then be empty.

### [LOW] liveHubGeometries has no test

**Location:** internal/slackcli/wiring.go:133
**Issue:** The live-hub seam is faked in every slackcli and slackengine test, so nothing runs the production function. D1 and D3 rely on its error carrying the hubs record's path, and the holder, keeper and login quote that error as it is. A change to the path in the message, or to the geometry built per hub, would pass every test. The rubric's test economy rule counts a behaviour the diff adds with no coverage as a finding.

### [NIT] Ack comment states a deadline the record says Slack does not name

**Location:** internal/slackengine/socket.go:32
**Issue:** The `Ack` comment says Slack asks for the acknowledgement within three seconds. The decision record's Slack facts say the docs name no Socket Mode acknowledgement deadline, and the holder waits ten seconds (`envelopeLimit`) before acknowledging a stalled route. The comment contradicts both.

## Rejected

- Lens code-affected-set, LOW: the `Pattern-Departure:` trailer on `fdc141fb` names an entry that does not forbid the way taken. Rejected: the entry's overview text says temp under `lymtemp`, so a temp file in the hub's Slack directory is a departure from its literal text. The trailer names a known entry, states the reason, and the decision record's D1 itself prescribes the temp file in `.lym/slack/`. A commit message cannot be fixed by a later edit.
- Lens code-tests, LOW: no test for a hub newly named by a `hubregistry.Live` read getting a resync hint. Rejected: `AFailingLiveHubsReadLeavesEventsUnackedAndNoticesOncePerEpisode` ends by adding a third hub to the live list and waiting for a resync hint in its inbox (`holder_test.go:809-813`).
- Handler check of the focus questions on the user token: a search of production code, docs and tests finds no `xoxp-` handling and no `unarchive` call. The only mentions are the test that forbids an unarchive method on the Slack seam and the login row that refuses an `xoxp-` token. `conversations.join` replaces the invite on `not_in_channel` for a post, a start read and a resync read, and on the `name_taken` adoption of a live channel.
- Handler check of the repository rules: `pattern/PATTERN-agent-addressing.md` names the bot and app-level tokens and has no user-token sentence, the `PATTERN.md` line already equals the declared text, `holder.go` is in the unbounded-wait list of `cmd/lym/daemonwakeups_test.go`, and `docs/slack-app.md` carries the manifest with `channels:join` and no user scope. `docs/overview.md` and both package docs describe the push route, the channel per orch and the two tokens. No change description exists at `_lym/landing/summary.md` to check.
