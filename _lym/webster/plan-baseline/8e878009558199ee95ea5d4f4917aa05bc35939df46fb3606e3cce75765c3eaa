# Card 6 — two-token-login

**Create:**
- `internal/slackengine#LiveHubs`
- `internal/slackcli#slackCLI.readTokens`
- `internal/slackcli#fileTokens`
- `internal/slackcli#liveHubGeometries`
- `internal/slackcli#fakeLoginSocket`
- `internal/slackcli#fakeLoginSocket.Read`
- `internal/slackcli#fakeLoginSocket.Ack`
- `internal/slackcli#fakeLoginSocket.Ping`
- `internal/slackcli#fakeLoginSocket.Close`
- `docs/slack-app.md`

**Delete:**
- `internal/slackcli#slackCLI.readToken`

**Edit:**
- `internal/slackcli#slackCLI`
- `internal/slackcli#slackCLI.loginCmd`
- `internal/slackcli#slackCLI.login`
- `internal/slackcli#tokenPrompt`
- `internal/slackcli#tokenRefusal`
- `internal/slackcli#botScopes`
- `internal/slackcli#noTokenMessage`
- `internal/slackcli#slackChecks`
- `internal/slackcli#wireProduction`
- `internal/slackcli#harness`
- `internal/slackcli#newHarness`
- `internal/slackcli#newFakeSlack`
- `internal/slackcli#slackRequest`
- `internal/slackcli#TestSlackCLI`
- `pattern/PATTERN-agent-addressing.md`
- `PATTERN.md`

**Uses:**
- `internal/slackengine#Credentials`
- `internal/slackengine#ReadCredentials`
- `internal/slackengine#WriteCredentials`
- `internal/slackengine#Identity`
- `internal/slackengine#API.BotAppID`
- `internal/slackengine#Dialer`
- `internal/slackengine#NewDialer`
- `internal/slackengine#HelloTimeout`
- `internal/hubregistry#Live`
- `internal/hubregistry#RecordFileName`
- `internal/standalonestate#UserStateDir`
- `internal/slackengine#StopRelay`

**Intent:**
Log the bot and app-level tokens in through one `lym slack login` run, writing both into the credential record card 5 adds (D3 as the operator-added entry overrides it: no user token).

`LiveHubs` is the engine seam that lists the machine's live hubs as their Slack geometries; login uses it here and the relay from card 9 on.

`login` reads two tokens in one run: on a terminal it prints `tokenPrompt` and reads one hidden line twice, through the existing `readHiddenLine` seam;
otherwise it reads standard input to its end and splits it at white space.
`readTokens` refuses an interrupted read or an empty line as `readToken` did, its messages naming the two kinds.
`fileTokens` files each token by its prefix, `xoxb-` as the bot token and `xapp-` as the app-level token.
It refuses any other prefix, and a run that does not give exactly one token of each kind, naming the two kinds and `lym slack login`; it writes nothing then.
Login then confirms the bot token with `auth.test`, reads the bot's app id with `bots.info` for the `bot_id` it answered,
and opens a Socket Mode connection with the app-level token through the new `slackCLI` seam `newDialer func(baseURL, appToken string) slackengine.Dialer`, closing it at once.
An event Slack hands that connection meanwhile goes unacknowledged, so Slack retries it.
A hello app id other than the bot's refuses the login naming the bot token's app and `lym slack login` with an app-level token of that app, and writes neither token.
A refused `auth.test` or `apps.connections.open` goes through `tokenRefusal`, now naming which kind Slack rejected.
Login writes both tokens with the app id through `WriteCredentials` once both are confirmed.
When a token it replaces was stored and differs, it ends the relay of each hub the new `slackCLI` seam `liveHubs` lists, through the existing `stop` seam with this process spared,
and the answer names each such hub under `relays_ended` with `lym slack start` as the way forward.
A failed live-hub read after the write answers an error saying the tokens stand, naming the seam's error,
and naming as the way forward `lym slack stop` then `lym slack start` in each hub of the machine, since a second login with the same tokens changes nothing and ends no relay.
The bound: login ends only relays of hubs the machine record lists, so a hub absent from it keeps an older relay until its own `lym slack stop`.
`liveHubGeometries` reads `hubregistry.Live` in `standalonestate.UserStateDir()` and builds each hub's `hubgeom.SlackGeometry`.
A failed `Live` read is returned wrapped as `slack: read the machine record of hubs <dir>/hubs.json: %w`, which keeps hubregistry's own message in the chain:
an undecodable record already names the file and the way forward of removing it, after which `lym creel inbox` in each hub records it again;
any other failed read carries the path and the read's own error alone.
A hub whose geometry fails is an error naming that hub's path and the geometry's own error, with the way forward that error names, if any.
Both are propagated errors no row reaches, and the socket holder's notice of card 9 and the keeper's of card 10 quote the seam's error as it is.
`wireProduction` fills `newDialer` with a closure `func(baseURL, appToken string) slackengine.Dialer { return slackengine.NewDialer(baseURL, appToken, &http.Client{}, slackengine.HelloTimeout) }`, the client built as `newSlack`'s is,
and `liveHubs` with `liveHubGeometries`; `slackChecks` gains `BotAppID`.
`noTokenMessage` names the two kinds.
`botScopes` and `loginCmd`'s help name the manifest's bot scopes, `channels:history`, `channels:join`, `channels:manage`, `channels:read`, `chat:write`, `chat:write.customize` and `users:read`,
the app-level scope `connections:write`, and `docs/slack-app.md`.

`docs/slack-app.md` (new, D10 with the operator-added entry) holds the app manifest and the steps to make a machine's app:
the app name `lym-<machine>` with Socket Mode on, the bot event `message.channels`, those bot scopes and the app-level token scope, and no user scope;
then installing the app, making an app-level token with `connections:write`, and running `lym slack login` with both tokens.

`pattern/PATTERN-agent-addressing.md`'s Slack boundary rule becomes: only `internal/slackengine` calls Slack, over the Web API and Socket Mode, reading its bot and app-level tokens only from `slack.json` in githubclient's credential directory, never from a config layer, `lym config env`, an environment variable or a repository; operator Slack text reaches an agent only as creel mail from `<shortname>:SLACK` of what Slack returned, and an agent reaches Slack only by creel mail to that program.
The overview line keeps its text, prescribed here whole as the declared change:

```markdown
- `PATTERN-agent-addressing` — Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; across hubs daemon to daemon with the sender confirmed, authority only for a trusted hub's orch. (test) — [background](pattern/PATTERN-agent-addressing.md)
```

Tests, tier0:
`TestSlackCLI`'s login rows: piped `xoxb-good xapp-good` stores both with app id `A1` after `/auth.test`, `/bots.info` and one dial;
on a terminal two prompts store the two hidden lines; an `xoxp-` token, two bot tokens and a lone bot token are each refused naming both kinds with no Slack call and no file;
a dial answering app `A2` is refused naming `lym slack login` and writes no file; a refused `auth.test` names the OAuth page and leaks no token;
a login replacing a stored bot token calls `stop` once per live hub and names both hubs with `lym slack start`, while a first login and a login repeating the stored tokens call none;
a replacing login whose `liveHubs` read fails writes both tokens and answers an error naming the seam's error, `lym slack stop` and `lym slack start`;
a refused `apps.connections.open` is refused naming the app-level token, a refused `bots.info` naming the bot token, and a dial failing on a transport error naming the network, each writing no file.
The piped row also asserts that `/bots.info` received `bot=B1`, the `bot_id` the `/auth.test` default answer carries.
The setup row `MissingScopeNamesTheScopes` now expects the manifest's bot scopes (`channels:history`, `channels:join`, `channels:manage`, `channels:read`, `chat:write`, `chat:write.customize`, `users:read`) and `docs/slack-app.md` in its refusal, in place of `groups:history`.
`newHarness` gains a fake dial answering a `fakeLoginSocket` with app `h.appID` (default `A1`) or `h.dialErr`, counting its dials, and a `liveHubs` seam answering `h.liveHubs` or `h.liveHubsErr`;
`fakeLoginSocket` answers nothing and records its `Close`.
`slackRequest` gains the request's form, which `newFakeSlack` records; `newFakeSlack` answers `/auth.test` with `bot_id` `B1` and `/bots.info` with app `A1` by default.

**ImpactSummary:** confined to `internal/slackcli`'s login verb and its test, the new `LiveHubs` seam type, `docs/slack-app.md` and the PATTERN background's Slack rule; login gains two seams and ends live relays only on a changed token.
