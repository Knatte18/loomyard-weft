# Card 6 — setup-operator-status

**Create:**
- `plan:internal/slackengine#SetOperator` -> `func SetOperator(path, operator string) error`

**Delete:**
- `internal/slackcli#channelRefusal`
- `internal/slackengine#API.ConversationInfo`
- `internal/slackengine#conversationInfoAnswer`

**Edit:**
- `internal/slackcli#slackCLI.setupCmd`
- `internal/slackcli#slackCLI.setup` -> `func (c *slackCLI) setup(ctx context.Context, operator string) (map[string]any, error)`
- `internal/slackcli#slackChecks`
- `internal/slackcli#slackCLI.statusCmd`
- `internal/slackcli#slackCLI.status`
- `internal/slackcli#slackCLI.command`
- `internal/slackcli#setupArgs`
- `internal/slackcli#newHarness`
- `internal/slackcli#newFakeSlack`
- `internal/slackcli#TestSlackCLI`
- `internal/slackengine#Setup`
- `internal/slackengine#SetupFileName`
- `internal/slackengine#RelayState`
- `internal/slackengine/template.yaml`
- `internal/slackengine_test#TestAPI`
- `internal/slackengine_test#documentedAnswers`
- `internal/slackengine_test#TestCredentialFiles`

**Uses:**
- `plan:internal/slackengine#ReadChannelMap`
- `plan:internal/slackengine#ChannelMap`

**Intent:**
Reduce `lym slack setup` to the operator and make `lym slack status` list the hub's channels (D10).

`setup --operator <user-id>` drops the `--channel` flag and the channel membership check: it confirms the user with `users.info` and records it with `SetOperator`.
`SetOperator` is a read-modify-write of `setup.json` under its `internal/state` lock that sets `operator_id` and keeps every other key,
so a `setup.json` an older binary wrote keeps its `channel_id`, which only card 12's missing-token line reads.
`Setup.ChannelID`'s comment says so; the field and its key stay.
`Setup`'s and `SetupFileName`'s comments name the operator record, and `template.yaml`'s comment says the operator, the channels and the tokens are never config, since `lym slack setup`, the relay and `lym slack login` write them.
`slackChecks` loses `ConversationInfo`, and `API.ConversationInfo` with its answer type goes, since nothing else calls it.
The group's help and `setupCmd`'s say lym makes one channel per orch and setup names only the operator; a running relay takes up a new operator at its next wake (card 12), with no restart.

`status` drops the `channel` field and prints `channels`: one object per map entry, sorted by orch, with `orch`, `name`, `id` and `archived`, omitted for an empty map;
an unreadable map is the status's error.
It prints `last_healthy` in place of `last_slack_ok`, read from `RelayState.LastSlackOK`, whose comment now names it the relay's last-healthy time (D1): card 12 stamps it at each Socket Mode pong and at the end of each outbound read.
The record key stays `last_slack_ok`, so a state an older relay wrote keeps feeding the restart cap.

Tests, tier0:
`TestSlackCLI`'s setup rows: `setupArgs` is `setup --operator U1`; `RecordsTheOperatorSlackConfirms` calls only `/users.info` and stores the operator;
a new row stores the operator over a `setup.json` holding `{"channel_id":"C9"}` and the file still holds `C9`;
the `BotNotAMember` and `ChannelNotFound` rows go with the check they reached; `MissingScopeNamesTheScopes` answers `/users.info` with `missing_scope` and names the manifest scopes and `docs/slack-app.md`; `UserNotFound` and `NoTokenNamesLogin` stay;
`--channel` is refused as an unknown flag.
The status row `PrintsPresenceRelayPauseSetupAndLastSlackCall`, renamed `PrintsPresenceRelayPauseOperatorChannelsAndLastHealthy`, writes a map with one open and one archived entry and expects `operator`, `channels` and `last_healthy`.
A new status row over a map file holding `{` is refused naming the map's path and moving it aside.
`newHarness`'s geometry sets `ChannelMapPath`; `newFakeSlack` drops its `/conversations.info` default.
`TestAPI` drops `ConversationInfo` and its documented answer; `TestCredentialFiles`'s setup row adds a `SetOperator` over a file with `channel_id` keeping it.

**ImpactSummary:** confined to `internal/slackcli`'s setup and status verbs and their test, plus `Setup`'s and `RelayState`'s comments and one removed client method whose only callers this card removes.
