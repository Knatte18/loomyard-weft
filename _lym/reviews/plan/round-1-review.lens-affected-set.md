```yaml
lens: affected-set
outcome: reported
```

Task: the affected-set lens. For each output, key, string or shape a card changes, find what reads or asserts it, and which cards cover that set.

**Per change: the set found, and the cards that cover it**

- **`RequireConfigured` re-sign and two-token refusal.**
  - Set: `start`, `away`, `runRelay` (`wiring.go:235`), `RunRelay` (`relay.go:186`), `TestCredentialFiles` (`files_test.go:113`), `TestRunRelay` (`relay_test.go:165`), `StartAndAwayNeedLoginAndSetup` (`cli_test.go:581`).
  - Covered by card 5.
- **`ReadToken`, `WriteToken` and `tokenRecord` deleted.**
  - Set: `starter.ensure`, `slackCLI.setup`, `slackCLI.login`, `TestCredentialFiles`, slackengine `newHarness` (`inbound_test.go:443`), `TestRunRelay` (`relay_test.go:174`), and slackcli `configure` and the `TestSlackCLI` rows (`cli_test.go:254, 373, 400, 455, 490, 588`).
  - Also `TestHubDobbyDeps_PrimeTickEnsuresRelay` (`beamcli/dobbydeps_integration_test.go:123-135`), whose line 135 sits inside that function.
  - Covered by card 5.
- **`AuthTest` re-sign.** Set: `slackChecks`, `slackCLI.login`, `TestAPI`. Covered by card 1.
- **`API.ConversationInfo` deleted.** Set: `slackChecks`, `slackCLI.setup`, `TestAPI`, `documentedAnswers`, the `/conversations.info` default of slackcli `newFakeSlack`. Covered by card 6.
- **`API.PostMessage` removed and `SlackClient` widened.** Covered by card 7.
  - Set: `inbound.answer`, `outbound.reportStale`, `outbound.postMessages`, `outbound.refused`, `recordingSlack`, `fakeSlack.PostMessage`, and the posting rows of `TestAPI`.
  - The only implementers are `API` and `fakeSlack` (grep `SlackClient`); no fake outside the package.
- **`Geometry` fields added (card 4); `CursorPath` and `CursorFileName` removed (card 10).**
  - Set: `hubgeom.SlackGeometry`, `TestSlackGeometry` (`hubgeom_test.go:479-511`), slackengine `newGeometry` (`inbound_test.go:411-418`), slackcli `newHarness` (`cli_test.go:166-173`), `harness.startAt` and `harness.cursor` (`inbound_test.go:466, 474`).
  - The literal at `files_test.go:116` sets only `TokenPath` and `SetupPath`, so neither card forces a change there.
  - Covered by cards 4, 6 and 10.
- **`RelayDeps` gains `LiveHubs` (card 10) and `Dial`/`Watch` (card 12).** Set: `wiring.go:243`, `relay_test.go:95`. Covered by cards 10 and 12.
- **Every `lym slack setup --channel` way forward.**
  - `credentials.go:87` is covered by card 5.
  - `inbound.go:228` and `inbound.go:297` (`readFailed`) are covered by card 10.
  - `outbound.go:81` is covered by card 11.
  - `cli.go:274, 286, 317, 332, 336, 340` (`setupCmd`, the membership check, `channelRefusal`) are covered by card 6.
  - No `.md` file under `docs/`, `contracts/`, `plugins/` or `pattern/` names `--channel`.
- **`botScopes` rewording (card 5).**
  - Set: `scopeRefusal` (`cli.go:360`), the `loginCmd` help (`cli.go:203-205`), and the row `MissingScopeNamesTheScopes` (`cli_test.go:481`), which asserts `groups:history`.
  - Card 6 covers that row; card 5 does not (finding 1).
- **`last_slack_ok` becomes `last_healthy` in status output.**
  - Set: `cli.go:563`, `cli_test.go:686-693` (`PrintsPresenceRelayPauseSetupAndLastSlackCall`).
  - No doc reads it; the record key stays.
  - Covered by card 6.
- **Status `channel` field replaced by `channels`.** Set: `cli_test.go:693`. Covered by card 6.
- **`(answer: @&lt;slug&gt; …)` hint and `[&lt;sender&gt;]` prefix removed.**
  - Set: `outbound.go:211-225`, `outbound_test.go:88-89`, `contracts/orchrole/role-slack.md:22-23`.
  - No test pins `role-slack.md`: no `_test.go` file names `role-slack` or `@&lt;slug&gt;`.
  - Covered by cards 11 and 12.
- **`readFailed` notice wording.** Set: `SlackRefusalMailsOneNoticePerEpisode` (`inbound_test.go:873`). Covered by card 10, which moves the row and renames it.
- **`tokenPrompt` and `tokenRefusal` wording.** Set: the `OnATerminal…` rows and `RefusedTokenNamesTheOAuthPageAndStoresNothing` (`cli_test.go:390-436`). Covered by card 5.
- **The Slack boundary rule in the `PATTERN-agent-addressing` background.** Set: `pattern/PATTERN-agent-addressing.md:23`. Covered by card 5.
- **The Slack loops in the `PATTERN-daemon-wakeups` background and the `unboundedWaitFiles` list.** Set: `pattern/PATTERN-daemon-wakeups.md` (Slack bullet), `cmd/lym/daemonwakeups_test.go:26-57`. Covered by cards 8 and 12.
- **The slack bullet and package lines of `docs/overview.md`.** Set: `docs/overview.md:391-392, 866-873`. Covered by card 12.
- **The `SlackGeometry` paragraph of `hubgeom/doc.go`.** Set: `internal/hubgeom/doc.go:23-26`. Covered by card 4.
- **Callers outside slack of `EnsureRelay`, `ReadPresence`, `LoadConfig` and `SlackProgram`.**
  - Set: `beamcli/dobbydeps.go:265-277`, `orchcli/start_test.go:1399`.
  - Their signatures and behaviour do not change for these callers, so no card needs to cover them.

**Findings**

1. **MEDIUM, class consistency, card 5 (`05-two-token-login.md`).**
   - Card 5 rewords `botScopes` to the manifest's scopes and drops `groups:history`. It lists `internal/slackcli#TestSlackCLI` as a target, but its Intent's test rows omit `MissingScopeNamesTheScopes`, which asserts `groups:history`.
   - Only card 6 rewrites that row. As planned, card 5's own tier0 gate fails on it.
   - The set's missed member is the `MissingScopeNamesTheScopes` row of `TestSlackCLI`.
   - Evidence: `internal/slackcli/cli_test.go:481`, `internal/slackcli/cli.go:31` and `:360`, card 5 Intent lines 353-354 and 372-383, card 6 Intent line 438.
   - Fix: card 5's Intent names that row's update, asserting the manifest scopes and `docs/slack-app.md`, so card 6's row change is a no-op or a re-statement.

Outside this lens, one note: card 6's Intent says card 12 stamps `LastSlackOK` at each pong and each outbound read, while cards 7 and 8 are the ones that stamp it.

Tool calls:
1. A grep of the repo for `last_slack_ok`, `answer: @`, `groups:history`, `xoxb`, `slackengine.Geometry{`, `RelayDeps{`, `ReadSetup(`, `TokenPath`, `SlackClient`, `slack status`, `cursor.json`, `conversations.history` and `slack setup`. The manifest held only the package-local greps, not the repo-wide readers of these outputs.
2. A read of the Geometry literal at `files_test.go:110-140`, the `func` lines of `dobbydeps_integration_test.go` and `hubgeom_test.go`, a search for test pins of the `role-slack` prose, and a grep of `docs/`, `contracts/`, `plugins/` and `pattern/` for slack strings. This placed the `WriteToken` and `CursorPath` call sites inside their enclosing test functions, which the manifest did not show.
3. A read of the `func` lines of `dobbydeps_integration_test.go` and `hubgeom_test.go` before lines 135 and 504, plus a grep for slackengine callers in beamcli, orchcli and fabriccli. This confirmed the functions enclosing those lines and the callers of slackengine outside the package.
