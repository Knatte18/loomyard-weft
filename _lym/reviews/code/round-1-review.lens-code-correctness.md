```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes with concrete inputs, including the failure of each call it makes.

**Cases run**

- `Store.post`, bridge nil, sender `ly:orch`, receiver `ly:SLACK`: `slackRelayCheck` returns nil, nil, and the post stores as before. Correct.
- `Store.post`, a refused sender: the error comes back before `interruptBeforePost` and `storePost`, so nothing is interrupted or stored. Correct.
- `Store.post`, live answer false, store succeeds: `storePost` sets `ReceiverLive` nil and no warning for a program receiver. Line 212 then overwrites them with `&false` and the relay-down warning. Correct.
- `Store.post`, live answer, `storePost` fails (for example, `ly:SLACK` is not yet a known program): `err != nil` skips the overwrite and the error is returned. Correct.
- `Store.post` with Wait: the server copies `posted.ReceiverLive` and `posted.Warning` into the `postStamp`. `awaitOutcome` writes the pending response with `receiver_live`, then the warning as a progress response (server.go:316, 471–476). Correct.
- `Store.post` with Interrupt to `ly:SLACK`: the bridge check runs first. A refusal returns before any interrupt. Otherwise `interruptBeforePost` → `strandTarget` refuses the program receiver, with nothing interrupted. Correct.
- `slackRelayCheck`, sender `Kind == KindProgram`: returns nil, nil. Correct.
- `slackRelayCheck`, EXTERNAL sender (`KindExternal`, seam.go:156): the bridge is asked. `IsOrchStrand("ly:EXTERNAL")` is false, so it is refused, which matches the relay dropping EXTERNAL (outbound.go `outboundRoute`). Correct, though the advice "ask your orch to post it" is odd for EXTERNAL.
- `slackRelayCheck`, `FormatProgram` error: swallowed and the check is skipped. It can only fail for an invalid shortname, which a running daemon's geometry never has. Correct in practice.
- `slackRelayCheck`, receiver spelled differently (`ly:slack`, `ly:pay:SLACK`): the comparison is exact. The relay reads only `<shortname>:SLACK` and `checkReceiverLocked` does no normalising (pool.go:261), so no spelling reaches the relay unchecked. Correct.
- `relayDownWarning`, live true gives an empty string; live false gives the text. See finding 2 on the wording.
- `RelayPostReady`, `ly:orch` with button on, relay lock held: returns true, nil. Correct.
- `RelayPostReady`, `ly:pay:orch-2`: `TaskSlug` gives `pay`, the orch is `ly:pay:orch`, and `hasButton` finds it through `beams`. Correct.
- `RelayPostReady`, `ly:orch-2`: `TaskSlug` gives "", so the orch is `ly:orch`. Correct.
- `RelayPostReady`, `ly:pay:driver`: refused as no orch. Correct.
- `RelayPostReady`, a pair orch `ly:zz:orch` with its forwarding on: refused as having no slack button. The relay does not drop such mail; see finding 3.
- `RelayPostReady`, failure of `geometry`, `beams`, `ForwardingOn` or `RelayLive`: the raw error is returned, so the post is refused with nothing stored. See finding 1.
- `RelayPostReady`, relay lock directory missing: `RelayLive` returns false, nil. Correct.
- `creelfake.Bridge.RelayPostReady`, `Refuse` and `SetRelayLive`: with nothing set, it answers live. The shared `refused` map has no cross-effect, because no existing test that calls `Refuse` posts to `ly:SLACK` (verbs_test.go:448 posts to machine addresses, machinemail_test.go:417 uses `Reply`). Correct.

**Findings**

1. **LOW**, `design`, `internal/creelcli/slackbridge.go:96-125` (`RelayPostReady` error paths). A failed read of the geometry, beam listing, forwarding file or relay lock now refuses the whole post to SLACK, with nothing stored and no `way forward:` clause. Before the diff these posts were stored and later relayed, so a transient read failure now loses mail the relay could still post. The board entry asks the post to report the relay state, not to fail closed on an unreadable state. Evidence: slackbridge.go:97-99, 109-111, 117-120 and 125, and verbs.go:230-233, where the error propagates and `storePost` never runs.

2. **LOW**, `design`, `internal/creelengine/verbs.go:243` (`relayDownWarning`). The warning says the message "waits in its mail until the relay starts". The relay counts mail older than `staleMailAge` (24h) as stale and never posts it, so a relay started after a day drops the post the warning promised would wait. Evidence: verbs.go:243, outbound.go:30 (`staleMailAge = 24 * time.Hour`), and the stale branch in the outbound loop (`record.Stale++`, which never posts).

3. **LOW**, `design`, `internal/creelcli/slackbridge.go:109-116` (`RelayPostReady` button rule). The bridge refuses an orch that is neither the hub's nor a listed beam's, and claims "the Slack relay drops its mail". The relay itself drops only on `outboundRoute` failure or a button that is off. For an orch whose forwarding is on but whose slug no beam lists, `channelFor` posts to the prime's channel instead of dropping. If such an orch can have its forwarding on, the refusal is wrong. Evidence: outbound.go `!sortedContains(forwarding, orch)` as the only button drop, and `channelFor`'s fallback "A beam the hub's beam listing does not name posts to the prime's channel".

Outside this lens, for the handler: the change description says the slack turn hook keeps its behaviour as a program sender. The hook actually posts as the orch strand (slackcli/hook.go:177-196, via `client.Post`), so it now goes through the bridge check.

Tool calls:
1. Read `staleMailAge`, `isOrchName`, the `ExternalSender` kind and `checkReceiverLocked`. My context had the call sites but not these definitions, which I needed to run the stale-mail, EXTERNAL and receiver-spelling cases.
