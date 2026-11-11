# Let two machines running lym talk to each other through Slack

`slack-machine-link` · Note · feature, slack

## Seen (operator, 2026-11-11)

Two machines run `lym`: OSL-1046 and Neve.
They have no direct way to talk to each other, and the operator wants none that needs more setup than it must.

## Idea

Use Slack as the middle man between two machines that run `lym`, so they can talk to each other with nothing else to set up.
The operator judges this more useful than the relay's mobile use alone.
Read first which messages should pass between the machines (the operator has not said whether tasks, status or plain messages), how a machine tells its own messages from the other's (the machine name, see `machine-name-file`), and how the existing Slack relay and login in `internal/slack*` are reused.
