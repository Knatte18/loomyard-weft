# Slack relay per hub: credentials, setup, inbound and outbound mail, presence and the lym slack verbs

`slack-relay` · Task · feature, reed

Depends on: [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md)

Filed from the Sketch `reed-slack-relay`.

Takes: creel's Go program client, given a read-with-wait wrapper in creelcli and an exported pane-ancestry walk in creelengine; runproc, selfexec, internal/state and lymdirs as they stand
Delivers: a live per-hub relay started by prime bring-up, prime dobby ticks and anchor changes; the token file, the hub's setup file and the presence state under .lym/slack/; a SLACK program mailbox orchs and lym programs post to; the slack config module with remind_after_min; the orch-role rules, written on top of creel-cross-hub-mail's landed sender and trust rules; and the two PATTERN changes under PATTERN.md's live cluster ids, as the Sketch's Constraints allow: PATTERN-reap-children's double-fork exception goes into PATTERN-child-processes and its background file, and PATTERN-slack-boundary joins PATTERN-agent-addressing (messages only through creel) as a change to it, since that cluster guards agent messaging
Exports: slackengine's presence reader for a hub (terminal or slack), the remind_after_min config loader, and the SLACK program name and receiver helper other packages post to

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
