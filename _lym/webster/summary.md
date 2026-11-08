# Slack relay with one push-woken channel per orch

All five batches recorded `done`.
A batch held several cards where the plan grouped them.

## Batches

- `01-api-web-methods` (cards 1–5), head `1342d37f`:
  the Web API calls, the Socket Mode client, channel names, the channel map and the credential record.
  Deviations: files under `internal/slackcli` and `internal/slackengine` outside the batch's own targets.
  No departures, no renames.
- `06-two-token-login` (cards 6–8), head `89dcc40b`:
  the two-token login, the operator-only setup with the map's channels in status, and the widened Slack seam.
  Deviations: `internal/slackcli`, `internal/slackengine/api.go`, `geometry.go`, `inbound.go` and their tests.
  Rename: `internal/slackengine#conversationInfoAnswer` became `createChannelAnswer`,
  because card 7 deletes the type while `CreateChannel` still decodes an answer of that shape.
  No departures.
- `09-socket-holder` (cards 9–10), head `07f62be7`:
  the socket holder and the channel keeper.
  Deviations: `internal/slackengine/export_test.go`, `holder.go`, `keeper.go` and their tests.
  Departure: `PATTERN-lym-files`, because the hint's temp file is created in the hub's Slack directory,
  outside `lymtemp`, so the rename into the inbox stays on one filesystem (named in the plan's Shared Decisions).
  No renames.
- `11-inbound-push` (cards 11–12), head `b7415eea`:
  inbound reads routed by channel, and outbound posts to the sender's channel.
  Deviations: `internal/slackengine/export_test.go`, `inbound.go`, `outbound.go`, `keeper_test.go` and their tests.
  No departures, no renames.
- `13-relay-push-loop` (card 13), head `321e2094`:
  the relay loop on push, with its wakes, retries and docs.
  Deviations: `internal/slackengine/export_test.go`, `relay.go`, `relay_test.go`.
  No departures, no renames.

## Scope guard

Every batch's `deviations` list names only files in the Slack packages (`internal/slackengine`, `internal/slackcli`).
The digests carried no warnings.
The forks reported the card and batch gates passing on tiers 0 and 1; tiers 2 and 3 were not run.
