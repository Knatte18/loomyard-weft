# Card 5 — two-token-login

**Create:**
- `plan:internal/slackengine#Credentials` -> `type Credentials struct`
- `plan:internal/slackengine#ReadCredentials` -> `func ReadCredentials(path string) (Credentials, error)`
- `plan:internal/slackengine#WriteCredentials` -> `func WriteCredentials(path string, creds Credentials) error`
- `plan:internal/slackengine#Credentials.Missing` -> `func (c Credentials) Missing() []string`
- `plan:internal/slackengine#LiveHubs` -> `type LiveHubs func() ([]Geometry, error)`
- `plan:internal/slackcli#slackCLI.readTokens` -> `func (c *slackCLI) readTokens(in io.Reader) ([]string, error)`
- `plan:internal/slackcli#fileTokens` -> `func fileTokens(tokens []string) (slackengine.Credentials, error)`
- `plan:internal/slackcli#liveHubGeometries` -> `func liveHubGeometries() ([]slackengine.Geometry, error)`
- `plan:internal/slackcli#fakeLoginSocket` -> `type fakeLoginSocket struct`
- `plan:internal/slackcli#fakeLoginSocket.Read` -> `func (s *fakeLoginSocket) Read(context.Context) (slackengine.Envelope, error)`
- `plan:internal/slackcli#fakeLoginSocket.Ack` -> `func (s *fakeLoginSocket) Ack(context.Context, string) error`
- `plan:internal/slackcli#fakeLoginSocket.Ping` -> `func (s *fakeLoginSocket) Ping(context.Context) error`
- `plan:internal/slackcli#fakeLoginSocket.Close` -> `func (s *fakeLoginSocket) Close() error`
- `docs/slack-app.md`

**Delete:**
- `internal/slackengine#ReadToken`
- `internal/slackengine#WriteToken`
- `internal/slackengine#tokenRecord`
- `internal/slackcli#slackCLI.readToken`

**Edit:**
- `internal/slackengine#RequireConfigured` -> `func RequireConfigured(geom Geometry) (Credentials, Setup, error)`
- `internal/slackengine#starter.ensure`
- `internal/slackengine#RunRelay`
- `internal/slackengine#TokenFileName`
- `internal/slackcli#slackCLI`
- `internal/slackcli#slackCLI.loginCmd`
- `internal/slackcli#slackCLI.login`
- `internal/slackcli#slackCLI.setup`
- `internal/slackcli#slackCLI.start`
- `internal/slackcli#slackCLI.away`
- `internal/slackcli#tokenPrompt`
- `internal/slackcli#tokenRefusal`
- `internal/slackcli#botScopes`
- `internal/slackcli#noTokenMessage`
- `internal/slackcli#slackChecks`
- `internal/slackcli#wireProduction`
- `internal/slackcli#runRelay`
- `internal/slackcli#harness`
- `internal/slackcli#newHarness`
- `internal/slackcli#newFakeSlack`
- `internal/slackcli#slackRequest`
- `internal/slackcli#harness.configure`
- `internal/slackcli#TestSlackCLI`
- `internal/slackengine_test#TestCredentialFiles`
- `internal/slackengine_test#newHarness`
- `internal/slackengine_test#TestRunRelay`
- `internal/slackengine_test#TestEnsureRelay`
- `internal/beamcli#TestHubDobbyDeps_PrimeTickEnsuresRelay`
- `pattern/PATTERN-agent-addressing.md`
- `PATTERN.md`

**Uses:**
- `plan:internal/slackengine#Identity`
- `plan:internal/slackengine#API.BotAppID`
- `plan:internal/slackengine#Dialer`
- `plan:internal/slackengine#NewDialer`
- `internal/hubregistry#Live`
- `internal/hubregistry#RecordFileName`
- `internal/standalonestate#UserStateDir`
- `internal/slackengine#StopRelay`

**Intent:**
Store the bot and app-level tokens in the machine's `slack.json` and log both in through one `lym slack login` run (D3 as the operator-added entry overrides it: no user token).

Engine:
`Credentials` holds `BotToken` under the existing `token` key, so an older binary still reads the bot token, `AppToken` under `app_token`, and `AppID` under `app_id`.
`ReadCredentials` reads the file through `internal/state`, a missing file reading as zero credentials;
`WriteCredentials` writes it atomically, creating the directory private to the user, as `WriteToken` did, and both replace `ReadToken`, `WriteToken` and `tokenRecord`.
`credentials.go`'s file header and `TokenFileName`'s comment name the two tokens and the app id.
`Missing` names each missing kind, `bot token (xoxb-)` and `app-level token (xapp-)`, in that order.
`RequireConfigured` now returns the credentials;
it refuses a machine missing either token, naming each missing kind and `lym slack login` from the operator's terminal with both tokens in one run, since login refuses a run with one,
and a hub with no setup file, naming `lym slack setup --operator <user-id>`, the wording `PATTERN-refusal-way-forward` and the record's Constraints ask for.
`starter.ensure` answers `OutcomeNotConfigured` with no error while either token or the setup file is missing, so `EnsureRelay` and `EnsureRelayWhileAway` spawn no relay that would refuse at once.
`RunRelay` takes the new result and still checks before the lock until card 12 moves the check after the first creel read.
`LiveHubs` is the seam that lists the machine's live hubs as their Slack geometries; login uses it here and the relay from card 8 on.

CLI:
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
Login writes both tokens with the app id once both are confirmed.
When a token it replaces was stored and differs, it ends the relay of each hub the new `slackCLI` seam `liveHubs` lists, through the existing `stop` seam with this process spared,
and the answer names each such hub under `relays_ended` with `lym slack start` as the way forward.
A failed live-hub read after the write answers an error saying the tokens stand, naming the seam's error,
and naming as the way forward `lym slack stop` then `lym slack start` in each hub of the machine, since a second login with the same tokens changes nothing and ends no relay.
The bound: login ends only relays of hubs the machine record lists, so a hub absent from it keeps an older relay until its own `lym slack stop`.
`liveHubGeometries` reads `hubregistry.Live` in `standalonestate.UserStateDir()` and builds each hub's `hubgeom.SlackGeometry`.
A failed `Live` read is an error naming the hubs record's path, `<dir>/hubs.json`, which the socket holder's notice of card 8 quotes;
a hub whose geometry fails is an error naming that hub's path and the geometry's own error, never the hubs record, so no notice sends the operator to move a sound record aside.
`wireProduction` fills `newDialer` with `slackengine.NewDialer` and `liveHubs` with `liveHubGeometries`; `slackChecks` gains `BotAppID`.
`setup` reads the bot token through `ReadCredentials`, and `noTokenMessage` names the two kinds.
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

Tests, all tier0 but the beamcli one:
`TestCredentialFiles` replaces its token rows with credentials reading back with private modes, the raw file still holding the bot token under `token`, and a missing file reading zero;
its `RequireConfigured` row covers no tokens, a bot token alone and an app-level token alone, each naming the missing kind and `lym slack login`, then no setup naming `lym slack setup --operator`.
`TestEnsureRelay`'s not-configured row adds a bot token alone answering `OutcomeNotConfigured` with no spawn.
`newHarness` and `TestRunRelay`'s no-token row write both tokens through `WriteCredentials`.
`TestSlackCLI`'s login rows: piped `xoxb-good xapp-good` stores both with app id `A1` after `/auth.test`, `/bots.info` and one dial;
on a terminal two prompts store the two hidden lines; an `xoxp-` token, two bot tokens and a lone bot token are each refused naming both kinds with no Slack call and no file;
a dial answering app `A2` is refused naming `lym slack login` and writes no file; a refused `auth.test` names the OAuth page and leaks no token;
a login replacing a stored bot token calls `stop` once per live hub and names both hubs with `lym slack start`, while a first login and a login repeating the stored tokens call none;
a replacing login whose `liveHubs` read fails writes both tokens and answers an error naming the seam's error, `lym slack stop` and `lym slack start`;
a refused `apps.connections.open` is refused naming the app-level token, a refused `bots.info` naming the bot token, and a dial failing on a transport error naming the network, each writing no file.
The piped row also asserts that `/bots.info` received `bot=B1`, the `bot_id` the `/auth.test` default answer carries.
`newHarness` gains a fake dial answering a `fakeLoginSocket` with app `h.appID` (default `A1`) or `h.dialErr`, counting its dials, and a `liveHubs` seam answering `h.liveHubs` or `h.liveHubsErr`;
`fakeLoginSocket` answers nothing and records its `Close`.
`slackRequest` gains the request's form, which `newFakeSlack` records; `newFakeSlack` answers `/auth.test` with `bot_id` `B1` and `/bots.info` with app `A1` by default; `configure` writes both tokens.
`StartAndAwayNeedLoginAndSetup` writes a bot token alone in its second step and expects the refusal naming the app-level token, then the setup refusal after both.
`TestHubDobbyDeps_PrimeTickEnsuresRelay` (tier1) writes both tokens with `WriteCredentials`.

**ImpactSummary:** replaces the token file API across `internal/slackengine`, `internal/slackcli` and one beamcli integration test; `RequireConfigured`'s re-sign reaches `RunRelay`, `start`, `away` and `runRelay`; login gains two seams and ends live relays only on a changed token.
