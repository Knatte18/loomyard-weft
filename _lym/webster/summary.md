# Slack relay per hub

All three batches recorded `done`, so the run ends `done`. No batch needed recovery, and no digest carried renames, departures or warnings.

## Batch 01-pattern-clusters (head `667e7cac`)

Covers cards 1 to 5.
It folds the double-fork exception into `PATTERN-child-processes` and the Slack boundary into `PATTERN-agent-addressing`.
It declares the Slack directory name, the relay's runproc role and githubclient's credential directory export, and exports creel's pane walk and orch test.
It codes the program-claim-in-strand-pane refusal and adds `ReadWait` and `HubStrands`.
It creates `internal/slackengine` with its geometry, token, setup, presence and relay-state files, hubgeom's teller, and the hub-wide `slack` config module holding `remind_after_min`.

Scope guard deviations (files outside the batch's own targets):
- `internal/configcli/configcli.go`
- `internal/slackengine/config.go`
- `internal/slackengine/credentials.go`
- `internal/slackengine/geometry.go`
- `internal/slackengine/presence.go`

## Batch 06-slackengine-api (head `6bd83089`)

Covers cards 6 to 9.
It adds the Slack Web API client with its failure kinds, the inbound poll with routing, presence commands and notices, and the outbound read that posts SLACK mail to the channel.
It also adds the relay's run loop, double-fork start, ensure, stop and waits.

Scope guard deviations:
- `internal/slackengine/api.go`
- `internal/slackengine/backoff.go`
- `internal/slackengine/export_test.go`
- `internal/slackengine/inbound.go`
- `internal/slackengine/outbound.go`
- `internal/slackengine/relay.go`
- `internal/slackengine/start.go`

## Batch 10-slack-cli (head `c7ac4f43`)

Covers cards 10 to 14.
It adds the `lym slack` module with its pane check and verbs, mounted in `newRoot`.
It starts the relay at the prime's bring-up and carries the presence in the resume pointer.
It ensures the relay on each prime dobby tick while presence is slack, starts it again when a prime anchor change ends, and adds the orch role's Slack rules.

Scope guard deviations:
- `internal/beamcli/dobbydeps.go`
- `internal/fabriccli/anchor.go`
- `internal/orchengine/prompt.go`
- `internal/orchengine/resumecontext_test.go`
- `internal/orchengine/seam_enforcement_test.go`
- `internal/slackcli/cli.go`
- `internal/slackcli/pane.go`
- `internal/slackcli/wiring.go`

## Departures and renames

`departures` and `renames` were empty in every digest, and `warnings` were null.
