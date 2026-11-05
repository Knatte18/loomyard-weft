# Card 6 — slackengine-api

**Create:**
- `internal/slackengine#DefaultBaseURL`
- `internal/slackengine#API`
- `internal/slackengine#NewAPI`
- `internal/slackengine#Message`
- `internal/slackengine#ChannelInfo`
- `internal/slackengine#APIError`
- `internal/slackengine#RateLimitError`
- `internal/slackengine#API.AuthTest`
- `internal/slackengine#API.ConversationInfo`
- `internal/slackengine#API.UserInfo`
- `internal/slackengine#API.History`
- `internal/slackengine#API.Latest`
- `internal/slackengine#API.PostMessage`
- `internal/slackengine/api_test.go`
- `internal/slackengine_test#TestAPI`

**Extends:**
- `internal/slackengine/doc.go`

**Intent:** Add the Slack Web API client over `net/http`, adding no dependency (Constraints), with the failure kinds the relay tells apart.

- `API` posts form-encoded requests to `<baseURL><method>` with `Authorization: Bearer <token>` and a bounded per-request timeout, and decodes `{"ok": …, "error": …}`.
- `AuthTest` calls `auth.test` (D2's token check); `ConversationInfo` calls `conversations.info` and answers `ChannelInfo{ID, Name, IsMember, IsPrivate}`; `UserInfo` calls `users.info`.
- `History` calls `conversations.history` with `oldest` (exclusive, Constraints) and `limit`, follows `response_metadata.next_cursor` while `has_more` is true, and returns every message oldest first, sorted by `ts` as Slack orders it newest first (D4).
  `Message` carries `TS`, `ThreadTS`, `User`, `Subtype`, `BotID` and `Text`.
- `Latest` calls `conversations.history` with `limit=1` and answers the newest `ts`, found false on an empty channel (D4's cursor start).
- `PostMessage` calls `chat.postMessage` with `channel` and `text`.
- Failure kinds: an HTTP `429`, or `ok: false` with `ratelimited`, is a `*RateLimitError` carrying `RetryAfter` from the `Retry-After` header in seconds (a missing header reads as one second); any other `ok: false` is an `*APIError` carrying Slack's error code, such as `msg_too_long`, `invalid_auth` or `channel_not_found`; a transport failure or a `5xx` is a plain wrapped error.
  The relay retries the latter two kinds without a cap and counts only `*APIError` toward D5's cap of 5.
- The token never appears in an error or a log line.

`internal/slackengine/api_test.go` is tier0, against an `httptest.Server` that records each request: `TestAPI` rows — each method sends its method path, the bearer header and its form fields; `History` over a fake with `has_more` across two pages returns both pages' messages oldest first and sends the second page's cursor; `Latest` on an empty channel answers found false; a `429` with `Retry-After: 7` is a `*RateLimitError` of 7 seconds; `ok: false` `msg_too_long` is an `*APIError` with that code; a `500` is neither; `ok: false` `invalid_auth` on `AuthTest` is an `*APIError` (D2's refused token).

**ImpactSummary:** New code inside the new package only; `doc.go` gains a paragraph on the client and its failure kinds.
