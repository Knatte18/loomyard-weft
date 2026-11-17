```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

No findings.

Each claim I checked holds:
- **Card 4:** `IsDaemonRefusal` is meant to match every refusal `Client.do` builds. It does: all three builders wrap in `&daemonRefusal{...}` at `internal/creelengine/client.go:274-278`. `daemonRefusal.Is` on the outer value matches `ErrDaemonRefusal`, and `Unwrap` still reaches the coded sentinels.
- **Card 5:** a `<machine>::<name>` sender fails `isStrandName`, because `isStrandName` is `agentname.Parse` succeeding (`internal/creelengine/sweep.go:175-178`). The new `Bound:` line agrees with `ErrProgramClaimInStrandPane`, which refuses only a program claim from a strand pane that is not an orch's (`internal/creelengine/identity.go:21-24`).
- **Cards 9 and 11:** `retryWindow` is an existing `slackengine` constant of five minutes (`internal/slackengine/inbound.go:33`). `inbound` has the Slack client, geometry and clock fields that card 9 says `runPeerCommand` reads (`in.slack`, `in.geom`, `in.clock` in `inbound.go:703-742`).
- **Allowlist keys:** every planned key is in a form the guard produces. The key builder drops the pointer receiver (`cmd/lym/testonlycode_linux_test.go:208-227`), so `internal/creelengine.machineOutboundEntry.message`, `internal/creelcli.GoClient.TakeMachine`, `internal/slackengine.bridgePoll.wake` and `internal/slackengine.API.BotUsersNamed` all match.
- **Card 10:** the quoted `doc.go` sentence matches `internal/slackengine/doc.go:79` word for word.

Tool calls:
- One Bash call read the `daemonRefusal` builders, `isStrandName`, the `retryWindow` const, the guard's receiver handling and the program-claim sentinel. My context did not hold these spans.
- One Read of `cmd/lym/testonlycode_linux_test.go:200-239` read `functionName`, to confirm how the guard keys a method. My context held only lines 1-90.
