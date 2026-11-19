```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes through concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- `creelengine.Store.post`, relay check placement (`verbs.go:193-196`): sender `ly:pay:driver` posting to `ly:SLACK`, with the bridge refusing it. The function returns `PostResult{}` and the error before `interruptBeforePost` and `storePost` run, so nothing is journaled or interrupted. Correct.
- `post` with `Interrupt` to `ly:SLACK` from an accepted orch: `slackRelayCheck` passes, and then `interruptBeforePost` → `strandTarget` refuses the program receiver, with nothing stored. Correct, and the order is harmless.
- `post` end (`verbs.go:212-214`), accepted orch with the relay down: `storePost` returns `ReceiverLive=nil` because `receiverLiveness` gives a program no liveness. It is then overwritten with `false` plus `relayDownWarning`. Correct.
- `post` end when `storePost` errors: the `err == nil` guard skips the stamp and the error is returned. Correct.
- `post` with `Wait`: `server.go:316` builds the stamp from `posted.ReceiverLive` and `posted.Warning`. `awaitOutcome` sends live in the pending response and the warning as progress (`server.go:471-475`). Correct.
- `post` with `err` reuse: `relayLive, err :=` declares `err`, the interrupt branch now assigns to it, and `posted, err :=` redeclares only `posted`. No shadowing bug. Correct.
- `slackRelayCheck` with a nil bridge: returns nil, nil. Correct.
- `slackRelayCheck` with a program sender (`KindProgram`): returns nil, nil. Correct, since the relay routes the hub's programs itself.
- `slackRelayCheck` with an EXTERNAL sender (`KindExternal`, name `ly:EXTERNAL`, `seam.go:151`): asks the bridge, which refuses because it is no orch strand. The relay also drops EXTERNAL (`outbound.go:227`). Correct.
- `slackRelayCheck` with receiver `ly:alpha`: no match, returns nil. Correct.
- `slackRelayCheck` when `FormatProgram` errors: the error is swallowed and the check skipped. `s.geom.Shortname` is validated when the store is built, so this path is unreachable. No defect.
- `slackRelayCheck` when the bridge errors: returns nil and the error. Correct.
- `relayDownWarning(r, true)` returns `""`. With `false` it returns text naming `lym slack start`. Correct.
- `slackBridge.RelayPostReady`, sender `ly:orch` with the button on and the relay lock held: `IsOrchStrand` is true, `Format("ly","",orch)` gives `ly:orch`, `hasButton` is true, `ForwardingOn` is true, and `RelayLive` gives true. Correct.
- `RelayPostReady`, thread `ly:orch-2`: `MatchesRole` is true and `TaskSlug` is `""`, so the orch is `ly:orch`. This matches `outboundRoute`. Correct.
- `RelayPostReady`, beam thread `ly:pay:orch-2`: resolves to `ly:pay:orch`, which is in the beam listing. Correct.
- `RelayPostReady`, pair orch `ly:zz:orch`: `hasButton` is false, so it is refused with a way forward. Correct.
- `RelayPostReady`, button off: refused with a way forward. Correct. The relay's drop at `outbound.go:198` matches.
- `RelayPostReady`, `agentname.Format` failure: unreachable, because `IsOrchStrand` already parsed the name. No defect.
- `RelayPostReady`, failure of `geometry()`, `beams()`, `ForwardingOn` or `RelayLive`: the post is refused with the raw error. See finding 1.
- `creelfake.Bridge.RelayPostReady`, `Refuse` and `SetRelayLive`: with the zero value, `relayDown` is false, so the relay reads as running. A refusal error wins over the relay state. Correct.

**Finding 1**
- severity: LOW
- class: design
- location: `internal/creelcli/slackbridge.go:96-125`
- summary: When the bridge cannot read its own inputs, an accepted orch's post to `<shortname>:SLACK` is refused. This covers a failed Slack geometry read, beam listing, forwarding read or relay-lock probe.
  - The refusal is a raw error that names no way forward and does not say the post was not stored.
  - Before this diff the post was stored, and the relay would still post it once it read the mail. For example, while the beam listing is unreadable the relay falls back to the channel map (`outbound.go:248-256`).
  - Only the relay-down answer needs `RelayLive`, so a failed probe could answer "cannot tell" (nil liveness) rather than refuse.
  - The Slack turn hook now also fails on these errors: `slackcli/hook.go:192` posts as the orch strand and logs "the reply is lost to Slack".
- evidence: `internal/creelcli/slackbridge.go:97-99,108-111,118-125`; `internal/creelengine/verbs.go:193-196`; `internal/slackengine/outbound.go:248-256`; `internal/slackcli/hook.go:192-194`

**Finding 2**
- severity: LOW
- class: design
- location: `internal/creelcli/slackbridge.go:108-115`
- summary: `hasButton` refuses a beam orch that the hub's beam listing does not name, as "has no slack button". The relay does not drop that mail: `channelFor` posts mail from an unlisted beam to the prime's channel (`outbound.go:260-266`). So the bridge refuses some posts the relay would have posted, and the refusal misstates why.
- evidence: `internal/creelcli/slackbridge.go:130-145`; `internal/slackengine/outbound.go:236-267`

Tool calls:
1. Read `internal/slackcli/hook.go:165-215` and `ExternalSender`. My context lacked the turn hook's sender kind and the EXTERNAL sender's name and kind, which decide whether `slackRelayCheck` runs for them.
2. Searched for `RelayLive` callers and slackengine lock uses. I needed to know whether the relay's own lock take could collide with the per-post probe.
3. Read the relay's lock acquisition and `start.go:190-215`. It showed the relay takes its lock with `AcquireWriteLockWithin` and a wait, so a brief probe collision does not stop it, and I raised no finding there.
