# Card 1 — api-web-methods

**Create:**
- `plan:internal/slackengine#Identity` -> `type Identity struct`
- `plan:internal/slackengine#File` -> `type File struct`
- `plan:internal/slackengine#Sender` -> `type Sender struct`
- `plan:internal/slackengine#API.BotAppID` -> `func (a *API) BotAppID(ctx context.Context, botID string) (string, error)`
- `plan:internal/slackengine#API.CreateChannel` -> `func (a *API) CreateChannel(ctx context.Context, name string) (ChannelInfo, error)`
- `plan:internal/slackengine#API.FindChannel` -> `func (a *API) FindChannel(ctx context.Context, name string) (ChannelInfo, bool, error)`
- `plan:internal/slackengine#API.Join` -> `func (a *API) Join(ctx context.Context, channel string) error`
- `plan:internal/slackengine#API.Invite` -> `func (a *API) Invite(ctx context.Context, channel, user string) error`
- `plan:internal/slackengine#API.Rename` -> `func (a *API) Rename(ctx context.Context, channel, name string) error`
- `plan:internal/slackengine#API.Archive` -> `func (a *API) Archive(ctx context.Context, channel string) error`
- `plan:internal/slackengine#API.Replies` -> `func (a *API) Replies(ctx context.Context, channel, thread, oldest string) ([]Message, error)`
- `plan:internal/slackengine#API.PostAs` -> `func (a *API) PostAs(ctx context.Context, channel, text string, sender Sender) error`
- `plan:internal/slackengine#API.OpenConnection` -> `func (a *API) OpenConnection(ctx context.Context) (string, error)`

**Edit:**
- `internal/slackengine#API`
- `internal/slackengine#NewAPI`
- `internal/slackengine#API.AuthTest` -> `func (a *API) AuthTest(ctx context.Context) (Identity, error)`
- `internal/slackengine#Message`
- `internal/slackengine#ChannelInfo`
- `internal/slackcli#slackChecks`
- `internal/slackcli#slackCLI.login`
- `internal/slackengine_test#TestAPI`
- `internal/slackengine_test#documentedAnswers`

**Uses:**
- `internal/slackengine#API.call`
- `internal/slackengine#decodeAnswer`
- `internal/slackengine_test#newFakeSlackAPI`
- `internal/slackengine_test#answerJSON`

**Intent:**
Add every Web API call the push route makes to `API`, each posting its form through `API.call` and decoding only the fields it reads, as the existing methods do.
No caller outside the client uses the new methods yet; later cards wire them.

- `AuthTest` now decodes `team_id`, `user_id` and `bot_id` into `Identity` (fields `TeamID`, `UserID`, `BotID`).
  Its only callers are `slackCLI.login`, which ignores the identity in this card, and the `slackChecks` interface, whose `AuthTest` method changes to the same result.
- `BotAppID` calls `bots.info` with `bot` and returns `bot.app_id` (bot scope `users:read`).
- `CreateChannel` calls `conversations.create` with `name` and `is_private=false` and returns the channel.
- `FindChannel` calls `conversations.list` with `types=public_channel`, `exclude_archived=false` and `limit=200`, following `response_metadata.next_cursor` until a page has none.
  It returns the channel whose `name` equals the name asked, found false when no page holds it.
- `Join` calls `conversations.join`; `Invite` calls `conversations.invite` with `channel` and `users`, and answers nil for the code `already_in_channel`.
- `Rename` calls `conversations.rename` with `channel` and `name`; `Archive` calls `conversations.archive`, and answers nil for `already_archived`.
- `Replies` calls `conversations.replies` with `channel`, `ts` and `oldest`, paginated like `History`, oldest first.
- `PostAs` calls `chat.postMessage` with `channel` and `text`, plus `username` and `icon_emoji` when `Sender` (fields `Username`, `IconEmoji`) sets them; like `PostMessage` it reads nothing past `ok`.
  `PostMessage` stays until card 8 moves its callers onto `PostAs`.
- `OpenConnection` calls `apps.connections.open` and returns the answer's `url`; the API it runs on is built with the app-level token.
- `Message` gains `LatestReply` (`latest_reply`) and `Files []File` (`files`, `File` holding `Name` from `name`).
- `ChannelInfo` gains `Creator` (`creator`) and `IsArchived` (`is_archived`).
- `API`'s and `NewAPI`'s comments, and `api.go`'s file header, say the client calls Slack with one token: the bot token, or the app-level token for `OpenConnection`.

Tests, tier0, in `TestAPI`: `documentedAnswers` gains one answer per new method, in the shapes Slack's method pages document, recorded here so no card reads docs.slack.dev:
- `/auth.test` gains `"bot_id": "B12345678"`, as a bot token's answer carries it;
- `/bots.info`: `{"ok":true,"bot":{"id":"B12345678","app_id":"A161CLERW","name":"beforebot","deleted":false}}`;
- `/conversations.create` and `/conversations.list`'s one page: `{"ok":true,"channel":{"id":"C0EAQDV4Z","name":"endeavor","is_archived":false,"creator":"U012A3CDE","is_member":true}}` and `{"ok":true,"channels":[{"id":"C012AB3CD","name":"general","is_archived":false,"creator":"U012A3CDE","is_member":true}],"response_metadata":{"next_cursor":""}}`;
- `/conversations.join`, `/conversations.invite` and `/conversations.rename`: `{"ok":true,"channel":{"id":"C012AB3CD","name":"general"}}`; `/conversations.archive`: `{"ok":true}`;
- `/conversations.replies`: `{"ok":true,"messages":[{"type":"message","user":"U061F7AUR","text":"island","thread_ts":"1482960137.003543","reply_count":1,"latest_reply":"1483037603.017503","ts":"1482960137.003543"},{"type":"message","user":"U061F7AUR","text":"one island","thread_ts":"1482960137.003543","parent_user_id":"U061F7AUR","ts":"1483037603.017503"}],"has_more":false}`;
- `/apps.connections.open`: `{"ok":true,"url":"wss://wss-primary.slack.com/link/?ticket=1234-5678"}`.

The row `EachMethodSendsItsRequestAndDecodesSlacksDocumentedAnswer` calls each new method and asserts its recorded form and decoded result.
Its `ConversationInfo` expectation gains the example's `Creator`.
New rows of that test: `FindChannel` follows two pages and finds the name on the second, and reports found false when no page holds it;
`Invite` answering `already_in_channel` and `Archive` answering `already_archived` return nil, while another code is an `*APIError`;
`PostAs` with an empty `Sender` sends no `username` or `icon_emoji`.

**ImpactSummary:** `AuthTest`'s re-sign reaches only `slackChecks` and `login` in `internal/slackcli`; every other change is additive inside `internal/slackengine`'s client and its one test.
