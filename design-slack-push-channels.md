# Slack relay: one Socket Mode connection per machine, one channel per orch, two tokens

`slack-push-channels` · Task · feature, slack, creel, orch · running · Drift-Baseline

Decision record: [Sketch](decision-slack-push-channels.md)

Filed from the Sketch `slack-v2`.

Takes: the per-hub relay of internal/slackengine as it stands, hubregistry.Live, hubgeom.SlackGeometry, fswatch.Watch's names filter, machinename and the slack-login-prompt token read landed as fd159ff2
Delivers: a relay that wakes on a Socket Mode hint and reads Slack after each channel's cursor; a channel map with one channel per orch that lym creates, and that the bot renames with a date suffix and archives only when its beam is gone, never reopening an archived channel and never unarchiving one on name_taken, so an orch needing Slack under a freed name gets a new channel; a bot that joins a channel by itself with conversations.join after not_in_channel or a name_taken adoption; inbound routing by channel and outbound routing by sender; a login that asks for the bot token (xoxb-) and the app-level token (xapp-) in one run, with no user token; lym slack stop archiving nothing; setup reduced to the operator; docs/slack-app.md; and the PATTERN-agent-addressing and PATTERN-daemon-wakeups backgrounds rewritten for the push route
Exports: the live-hubs seam and the Socket Mode connection seam on slackengine.RelayDeps, the channel map record, the two-token credential record, RequireConfigured's missing-kind refusal, and the relay state's last-healthy stamp read by EnsureRelayWhileAway's restart cap; EnsureRelay and EnsureRelayWhileAway keep their signatures for orchcli, fabriccli and beamcli

The part changes these packages:

- internal/slackengine (about 4500 changed lines)
- internal/slackcli (about 700 changed lines)
- internal/hubgeom (about 40 changed lines)
- cmd/lym (about 20 changed lines)
- pattern (about 20 changed lines)
- docs (about 110 changed lines)
