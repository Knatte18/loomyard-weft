# Relevant files — creel-cross-hub-mail

- `internal/creelengine/doc.go` — the daemon, journal, sender identity, receivers, reply and retention rules the cross-hub route extends
- `internal/creelengine/identity.go` `resolveSender`, `checkProgramClaim`, `paneOfPeer` — how the sending daemon knows its sender, and why a program claim carries no trust
- `internal/creelengine/seam.go` `Sender`, `SenderKind`, `isOrchName`, `Sender.String` — the sender kinds, the orch check D5 uses and where the stamp is rendered
- `internal/hubgeom/creelgeom.go` `CreelGeometry` — builds another hub's creel geometry from its path for D2 and D3
- `internal/shuttleengine/claudeengine/settings.go` `creelTrustSentence` — the trust sentence D7 rewrites
- `contracts/stencils/parent/parent-directive-parent.md` — the delegate text D7 extends
- `contracts/orchrole/orch-role.md` — the orch's mail rules D7 extends
- `internal/standalonestate/standalonestate.go` `Derive` — the per-user lym state directory D1's record sits in
- `pattern/PATTERN-agent-addressing.md` — the entry the record's PATTERN change edits
- `internal/reedcli/add.go` — `--role orch` numbers beside a live orch and `--name` gives any role, which shapes D5's exact orch names
- `pattern/PATTERN-config-layers.md` — categories and layers for the `creel` module
