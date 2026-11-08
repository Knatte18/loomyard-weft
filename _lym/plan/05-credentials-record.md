# Card 5 — credentials-record

**Create:**
- `internal/slackengine#Credentials`
- `internal/slackengine#ReadCredentials`
- `internal/slackengine#WriteCredentials`
- `internal/slackengine#Credentials.Missing`

**Delete:**
- `internal/slackengine#ReadToken`
- `internal/slackengine#WriteToken`
- `internal/slackengine#tokenRecord`

**Edit:**
- `internal/slackengine#RequireConfigured` -> `func RequireConfigured(geom Geometry) (Credentials, Setup, error)`
- `internal/slackengine#starter.ensure`
- `internal/slackengine#RunRelay`
- `internal/slackengine#TokenFileName`
- `internal/slackcli#runRelay`
- `internal/slackcli#slackCLI.login`
- `internal/slackcli#slackCLI.setup`
- `internal/slackcli#slackCLI.start`
- `internal/slackcli#slackCLI.away`
- `internal/slackcli#harness.configure`
- `internal/slackcli#TestSlackCLI`
- `internal/slackengine_test#TestCredentialFiles`
- `internal/slackengine_test#newHarness`
- `internal/slackengine_test#TestRunRelay`
- `internal/slackengine_test#TestEnsureRelay`
- `internal/beamcli#TestHubDobbyDeps_PrimeTickEnsuresRelay`

**Uses:**
- `internal/slackengine#ReadSetup`

**Intent:**
Store the machine's Slack credentials as one record of the bot token, the app-level token and the app id, and refuse a machine missing either token (D3 as the operator-added entry overrides it: no user token).
Card 6 makes `lym slack login` write both tokens.

`Credentials` holds `BotToken` under the existing `token` key, so an older binary still reads the bot token, `AppToken` under `app_token`, and `AppID` under `app_id`.
`ReadCredentials` reads the file through `internal/state`, a missing file reading as zero credentials;
`WriteCredentials` writes it atomically, creating the directory private to the user, as `WriteToken` did, and both replace `ReadToken`, `WriteToken` and `tokenRecord`.
`credentials.go`'s file header and `TokenFileName`'s comment name the two tokens and the app id.
`Missing` names each missing kind, `bot token (xoxb-)` and `app-level token (xapp-)`, in that order.
`RequireConfigured` now returns the credentials;
it refuses a machine missing either token, naming each missing kind and `lym slack login` from the operator's terminal with both tokens in one run, the form card 6's login takes,
and a hub with no setup file, naming `lym slack setup --operator <user-id>`, the wording `PATTERN-refusal-way-forward` and the record's Constraints ask for.
`starter.ensure` answers `OutcomeNotConfigured` with no error while either token or the setup file is missing, so `EnsureRelay` and `EnsureRelayWhileAway` spawn no relay that would refuse at once.
`RunRelay` takes the new result and still checks before the lock until card 13 moves the check after the first creel read.
`internal/slackcli`'s `runRelay` builds the API with the returned credentials' bot token, and `start` and `away` take the new result.

Until card 6, `login` reads and writes the bot token alone through the record:
it reads the stored credentials and replaces only `BotToken`, so a stored app-level token and app id stand.
`setup` reads the bot token through `ReadCredentials`.
So between this card and card 6 no production path writes an app-level token:
`start` and `away` refuse and `EnsureRelay` answers `OutcomeNotConfigured` on every machine until card 6's login writes both tokens.

Tests, all tier0 but the beamcli one:
`TestCredentialFiles` replaces its token rows with credentials reading back with private modes, the raw file still holding the bot token under `token`, and a missing file reading zero;
its `RequireConfigured` row covers no tokens, a bot token alone and an app-level token alone, each naming the missing kind and `lym slack login`, then no setup naming `lym slack setup --operator`.
`TestEnsureRelay`'s not-configured row adds a bot token alone answering `OutcomeNotConfigured` with no spawn.
`newHarness` and `TestRunRelay`'s no-token row write both tokens through `WriteCredentials`.
`internal/slackcli`'s `configure` writes both tokens.
`TestSlackCLI`'s rows that call `WriteToken` or `ReadToken` call `WriteCredentials` or `ReadCredentials`;
the login row `StoresTheTokenOnlyAfterSlackAcceptsIt` runs over stored credentials holding an app-level token and app id `A1`, and both still stand after the login.
`StartAndAwayNeedLoginAndSetup` writes a bot token alone in its second step and expects the refusal naming the app-level token, then the setup refusal after both.
`TestHubDobbyDeps_PrimeTickEnsuresRelay` (tier1) writes both tokens with `WriteCredentials`.

**ImpactSummary:** replaces the token file API across `internal/slackengine`, `internal/slackcli` and one beamcli integration test; `RequireConfigured`'s re-sign reaches `RunRelay`, `start`, `away` and `runRelay`; login and setup keep their behaviour over the new record.
