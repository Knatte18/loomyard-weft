# Card 9 — approve-and-revoke

**Create:**
- `plan:internal/slackengine#peerCommand` -> `type peerCommand struct`
- `plan:internal/slackengine#parsePeerCommand` -> `func parsePeerCommand(text string) (peerCommand, bool)`
- `plan:internal/slackengine#inbound.runPeerCommand` -> `func (in *inbound) runPeerCommand(ctx context.Context, command peerCommand) (string, error)`

**Edit:**
- `internal/slackengine#Hub`
- `internal/slackengine#inbound.handle`
- `internal/slackengine#inbound.status`
- `internal/slackengine_test#fakeHub`
- `internal/slackengine_test#newRelayRun`
- `internal/slackengine_test#TestInboundRead`
- `internal/slackcli#runRelay`

**Extends:**
- `internal/slackengine/doc.go`
- `docs/slack-app.md`

**Uses:**
- `internal/slackcli/doc.go`
- `plan:internal/slackengine#ReadPeers`
- `plan:internal/slackengine#UpdatePeers`
- `plan:internal/slackengine#Peer`
- `internal/slackengine#SlackClient`
- `internal/slackengine#ReadCredentials`
- `internal/slackengine#inbound.retryOrGiveUp`
- `internal/slackengine#inbound.answer`
- `internal/machinename#CheckName`
- `internal/machinename#Names`

**Intent:**
D6 in an orch channel: the operator's `approve <machine>`, `approve <machine> <app id>` and `revoke <machine>`, and the app id in `status`.

`Hub` gains `MachineNames func() (current string, former []string, err error)`; `runRelay` fills it with `machinename.Names`, and `fakeHub` and `newRelayRun`'s own `Hub` literal with fixed names, so no test builds a relay with it nil; production always sets it, so a nil field is a wiring fault that panics on use.
`parsePeerCommand` matches the command word case-insensitively and lower-cases the machine name; it answers `peerCommand{Verb, Machine, AppID}` for `approve <m>`, `approve <m> <app id>` (the app id kept as written) and `revoke <m>`, and false for any other text.
`inbound.handle` checks it after `away`, `back` and `status`, so it runs only for the configured operator's plain message, as those do.
In an orch channel, a machine that is this machine's current or a former name (`Hub.MachineNames`) is refused in the channel: `approve and revoke name another machine; type approve %s in a channel of that other machine`, naming this machine.
`runPeerCommand` does the work and answers the line the relay posts in the channel the command was written in:
- A machine failing `machinename.CheckName` is refused naming the rule.
- `approve <m>`: `BotUsersNamed(ctx, "lym-"+m)`. One candidate is pinned (`UpdatePeers` sets `Peer{AppID, BotUserID, ApprovedAt}`) and the answer names its app id. None or several are refused, listing each candidate's app id and `approve <m> <app id>`.
- `approve <m> <app id>`: it lists every bot user (`BotUsersNamed` with an empty name, which keeps every bot) and pins under m the one whose app id equals the given one exactly, whatever its name, so a renamed machine is approved this way.
  With none of that app id it refuses, naming the id, with the way forward: read the app id from `status` typed in a channel of the other machine.
- `revoke <m>`: removes the pin and says so; a machine with no pin is answered as such.
A Slack refusal or a record that fails to write goes through `retryOrGiveUp`, so a lasting failure is answered after five minutes and the channel moves on.
`status` answers `presence is %s; this machine's app id is %s`, the app id from `ReadCredentials(geom.TokenPath)`, so the operator compares it with the other machine's before approving; with no app id recorded it answers `presence is %s; this machine's app id is unknown, run lym slack login`.
The existing `TestInboundRead` rows that assert `presence is slack` and `presence is terminal` exactly are reworded to the new answer.
Bound: anyone signed in as the operator's Slack account can approve a machine, the authority that already instructs every orch; an approved app is trusted to name its sender, and an impostor app named `lym-<m>` before the real one is caught only by comparing app ids with `status`.

`TestInboundRead` (tier 0) gains rows over `fakeSlack`'s bot directory:
- `approve neve` with one `lym-neve` bot pins its app and bot user and answers its app id;
- with none, and with two, it answers listing the candidates' app ids and pins nothing;
- `APPROVE Neve A0UPPER` pins the bot with app id `A0UPPER` under `neve`, the id kept as written;
- `revoke neve` removes the pin;
- `approve` naming this machine's current or a former name is refused naming the command for the other machine;
- the same commands from a user who is not the operator are not handled and nothing is written;
- `status` names this machine's app id, and `unknown` with no app id recorded;
- `approve Bad_Name` is refused naming the name rule; `approve neve A0NONE` with no bot of that app id is refused naming the id and `status`; `revoke osl` with no pin is answered as having none;
- a `users.list` refusal from `fakeSlack` holds the message and is retried, and after five minutes of clock the relay answers it and moves on.
`docs/slack-app.md` gains a section on pairing two machines: both apps in one workspace, each machine's `status` to read its app id, and `approve <machine>`, or `approve <machine> <app id>`, typed on each machine, in an orch channel of that machine; the manifest stays as it is.

**ImpactSummary:** `inbound.handle` gains one command branch before forwarding and `status` one clause; `Hub` gains a field filled by `runRelay`, its one production builder, and by `fakeHub`.

**Commit:** 9: the operator approves and revokes peer machines in Slack
