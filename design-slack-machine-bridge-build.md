# Build the Slack bridge that carries creel mail between orchs on two machines

`slack-machine-bridge-build` · Task · feature, slack, creel, orch · draft

Decision record: [Sketch](decision-slack-machine-bridge-build.md)

Filed from the Sketch `slack-machine-bridge`.

Takes: the Sketch's decision record, creel's cross-hub path in internal/creelengine and the Slack relay in internal/slackengine as they stand
Delivers: orch-to-orch creel mail across two machines through a per-pair Slack bridge channel, gated by the operator's approve and revoke commands and the orchs' slack buttons, stamped (other machine, trusted orch) or untrusted on receipt
Exports: agentname's <machine>::<name> parser; creel's machine-outbound and machine-inbound events with the program-only take, report, settle and deliver verbs and their GoClient methods; slackengine's bridge envelope formatter and parser, bridge map, bridge-notice and bridge-post records, approved-peer record and bridge poll

The part changes these packages:

- internal/agentname (about 150 changed lines)
- internal/creelengine (about 1300 changed lines)
- internal/creelcli (about 350 changed lines)
- internal/slackengine (about 2000 changed lines)
- internal/slackcli (about 80 changed lines)
- internal/hubgeom (about 20 changed lines)
- internal/shuttleengine (about 10 changed lines)
- cmd/lym (about 10 changed lines)
- contracts/orchrole (about 30 changed lines)
- contracts/stencils/parent (about 3 changed lines)
- docs (about 30 changed lines)
- pattern (about 17 changed lines)
