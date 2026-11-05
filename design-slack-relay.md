# Slack relay per hub: credentials, setup, inbound and outbound mail, presence and the lym slack verbs

`slack-relay` · Task · feature, reed

Depends on: [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md)

Filed from the Sketch `reed-slack-relay`.

Takes: creel's Go program client, given a read-with-wait wrapper in creelcli and an exported pane-ancestry walk in creelengine; runproc, selfexec, internal/state and lymdirs as they stand
Delivers: a live per-hub relay started by prime bring-up, prime dobby ticks and anchor changes; the token file, the hub's setup file and the presence state under .lym/slack/; a SLACK program mailbox orchs and lym programs post to; the slack config module with remind_after_min; the orch-role rules; and the two PATTERN changes under PATTERN.md's live cluster ids, as the Sketch's Constraints allow: PATTERN-reap-children's double-fork exception goes into PATTERN-child-processes and its background file, and PATTERN-slack-boundary joins PATTERN-agent-addressing (messages only through creel) as a change to it, since that cluster guards agent messaging
Exports: slackengine's presence reader for a hub (terminal or slack), the remind_after_min config loader, and the SLACK program name and receiver helper other packages post to

## Fit with the cross-hub mail design (orch, operator 2026-11-05)

`creel-cross-hub-mail` is a motif: it lands its design as a Sketch (a stored decision record), not code, and this task waits on it for that design.
This run's Discussion reads that record (`lym board get <its slug or S handle> --record`) and designs the Slack sender and its authority to fit it, so the two never need reworking:

- How creel names and trusts a sender that is not one of the hub's own strands: `<shortname>:SLACK` beside the confirmed cross-hub sender and `EXTERNAL`.
- The orch-role rules on whose mail carries the operator's authority, written once so the cross-hub build later adds its trusted-orch rule beside the SLACK rule.
- Logging: every relayed message journaled in creel, as the cross-hub design journals every message and every drop.

The cross-hub design's build tasks, filed later by raddle from that Sketch, run after this task and build on its SLACK sender.

The part changes these packages:

- internal/slackengine (about 1600 changed lines)
- internal/slackcli (about 700 changed lines)
- internal/creelcli (about 60 changed lines)
- internal/creelengine (about 15 changed lines)
- internal/hubgeom (about 25 changed lines)
- internal/orchcli (about 30 changed lines)
- internal/orchengine (about 25 changed lines)
- internal/fabriccli (about 20 changed lines)
- internal/beamcli (about 20 changed lines)
- internal/githubclient (about 15 changed lines)
- internal/runproc (about 5 changed lines)
- internal/lymdirs (about 5 changed lines)
- internal/configreg (about 5 changed lines)
- cmd/lym (about 10 changed lines)
- contracts/orchrole (about 15 changed lines)
- pattern (about 30 changed lines)
