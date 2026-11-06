# Creel mail between hubs: a machine record of hubs, confirmed daemon-to-daemon delivery and replies, per-hub trust and the sender stamp

`creel-cross-hub-build` · Task · feature, creel, orch · running · Plan-Burler-Review

Filed from the Sketch `creel-cross-hub-mail`.

Takes: creel's journal, fold, delivery seam and verbs, and the orch role, as slack-relay leaves them, with its <shortname>:SLACK program sender and its orch-role rules on whose mail carries the operator's authority; fabricengine.ReadShortname, hubgeom.CreelGeometry, internal/state and internal/lock as they stand
Delivers: the per-user hubs.json machine record written by each creel daemon at start; posts and replies from a strand to another hub's strand, queued in the sending journal, delivered daemon to daemon and confirmed with the sending daemon, with drops journaled and noticed to the orch; the creel config module with trusted_hubs; the sender stamp on every rendering of mail from another hub; the trust sentence, parent directive and orch role reading the stamps; lym creel hubs; the PATTERN-agent-addressing change; and the board note creel-cross-hub-trial
Exports: hubregistry's record and lookup by shortname, creelconfig's trusted_hubs loader, the deliver and confirm verbs of the creel socket protocol, and the lym creel hubs verb

The part changes these packages:

- internal/creelengine (about 2500 changed lines)
- internal/creelcli (about 300 changed lines)
- internal/hubregistry (about 300 changed lines)
- internal/creelconfig (about 120 changed lines)
- internal/configreg (about 5 changed lines)
- internal/shuttleengine (about 5 changed lines)
- internal/shuttleengine/claudeengine (about 5 changed lines)
- contracts/stencils (about 10 changed lines)
- contracts/orchrole (about 15 changed lines)
- cmd/lym (about 10 changed lines)
- pattern (about 10 changed lines)
