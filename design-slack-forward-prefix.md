# Slack messages to the orch say Slack twice: sender ly:SLACK and a [slack] body prefix

`slack-forward-prefix` · Task · enhancement, slack · draft

## Seen (operator, 2026-11-08)

A Slack message reaches the orch as `[creel] #1033 from "ly:SLACK": [slack] OK. ...`: the sender `ly:SLACK` already says it came from Slack, and the body opens with `[slack] ` again (`forwardPrefix` in `internal/slackengine/inbound.go`).

## Idea

Drop the `[slack] ` body prefix, or keep it only where the sender does not show, once checked that no reader keys on it.
Built after the Slack relay rebuild `slack-push-channels` lands, since that task rewrites inbound routing in the same package and the operator holds Slack work until then.
