# Relevant files

- `internal/slackengine/doc.go` — the relay's whole behaviour: files, inbound, outbound, holder, wake loop
- `internal/slackengine/inbound.go` `handle`, `forward` — operator-only forwarding and the presence commands the approve commands join
- `internal/slackengine/api.go` `BotAppID`, `Invite`, `FindChannel`, `PostAs` — the Slack calls the bridge reuses
- `internal/slackengine/keeper.go` — channel creation, adoption by creator and archiving
- `internal/slackengine/channelname.go` `ChannelName` — sanitizing and the 80-character cut the bridge name reuses
- `internal/agentname/agentname.go` `FormatMachineAddress`, `MachineAddressSeparator` — the display form the new parser reads back
- `internal/machinename/machinename.go` `Name`, `CheckName` — this machine's name and the valid machine-name form
- `internal/creelengine/doc.go` — the cross-hub mail section the machine path mirrors
- `internal/creelengine/crosshub.go` — posting to another hub's strand, journaled outbound
- `internal/creelengine/receive.go` — inbound delivery, reply-to check, trust decision
- `internal/creelengine/fold.go` `HubStamp` — where the stamp is chosen
- `internal/creelcli/hubdirectory.go` — creel's wiring to the machine record, where the bridge seam is filled
- `contracts/orchrole/role-slack.md` — the orch's Slack instructions to extend
- `docs/slack-app.md` — the app manifest and setup steps
- `pattern/PATTERN-agent-addressing.md` — the across-hubs and Slack boundary rules the design extends
- `pattern/PATTERN-daemon-wakeups.md` — the relay loops list the bridge poll joins
