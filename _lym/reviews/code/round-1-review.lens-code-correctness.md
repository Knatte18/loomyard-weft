```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced every function and branch the diff adds or changes, using concrete inputs.

**Cases run**
- `slackRelayCheck`, nil bridge: answers nil, nil, and `post` stores the post with no liveness. Correct.
- `slackRelayCheck`, program sender `ly:BATTEN`: answers nil, nil, and the bridge is never asked. Correct, and it matches the relay posting a program's mail.
- `slackRelayCheck`, receiver `ly:BEAM` (not the relay): answers nil, nil. Correct.
- `slackRelayCheck`, receiver spelled differently, such as `SLACK` or `zz:SLACK`: no match, so it answers nil. `checkReceiverLocked` (pool.go:265) then accepts only an exact registered program name, so no other spelling reaches the relay's mail. Correct.
- `slackRelayCheck`, `FormatProgram` fails: the error is swallowed and the check is skipped. It fails only on an invalid store shortname, and then no receiver can equal the relay name. Harmless.
- `slackRelayCheck`, bridge refuses: the refusal is returned before `interruptBeforePost` and `storePost`, so nothing is journaled or interrupted. Correct.
- `post`, `--interrupt` to `ly:SLACK`: the bridge is asked first. `strandTarget` then refuses a program receiver, so nothing is stored. Correct.
- `post`, `storePost` fails: `err != nil`, so the relay state is not applied and the error is returned. Correct.
- `post`, stored, relay down: `ReceiverLive=false` and the warning is set, overwriting the nil from `receiverLiveness`. Correct.
- `post`, stored, relay live: `ReceiverLive=true` and the warning is empty. Correct.
- Daemon, `Wait` post: the pending response carries `receiver_live` and the warning goes out as a progress response (server.go:471-476). The CLI prints it through `warn` (cli.go:452-456). Correct.
- `relayDownWarning`: live gives ""; down gives a text naming the receiver and `lym slack start`. Correct.
- `RelayPostReady`, EXTERNAL sender `ly:EXTERNAL`: `IsOrchStrand` is false, so it is refused as "no orch". This matches `outboundRoute` dropping EXTERNAL (outbound.go:228). Correct.
- `RelayPostReady`, beam orch thread `ly:pay:orch-2`: `TaskSlug` gives "pay", which forms `ly:pay:orch`, the same mapping `outboundRoute` uses. Correct.
- `RelayPostReady`, prime thread `ly:orch-2`: the slug is "", which forms `ly:orch`. Correct.
- `RelayPostReady`, `Format` or `TaskSlug` failure: unreachable. `IsOrchStrand` already parsed the name, and a strand sender is of this hub's shortname. Correct.
- `RelayPostReady`, pair orch `ly:zz:orch`: `hasButton` is false, so it is refused. The toggle refuses a pair's orch a button (toggle.go:173), so the relay would drop its mail too. Correct.
- `RelayPostReady`, button off: refused naming the status bar. Correct.
- `RelayPostReady`, button on: answers `RelayLive(geom)`. Correct.
- `RelayPostReady`, beam listing fails for a beam orch: `hasButton` returns the error, so the post is refused and nothing is stored. Not correct; see finding 2.
- `RelayPostReady`, `ForwardingOn` fails or `RelayLive` errors: the post is refused with a bare error. Not correct; see finding 2.
- `Reply` (`post --reply &lt;id&gt;`) to mail from `ly:SLACK`: it appends a reply to `ly:SLACK` through `appendPostedLocked` (verbs.go:399) and never reaches `post` or `slackRelayCheck`. Not correct; see finding 1.
- `creelfake.Bridge.RelayPostReady`: answers a refusal by name, else `!relayDown`. Correct.
- `creelfake.Bridge.SetRelayLive`: correct.

**Findings**

1. **MEDIUM, design.** `internal/creelengine/verbs.go:350` (`Reply`). A reply to the operator's Slack mail skips the relay check entirely.
   - `post --reply &lt;id&gt;` to a message from `ly:SLACK` stores the reply through `appendPostedLocked` (verbs.go:399) with no `RelayPostReady` call, no `receiver_live` and no warning.
   - A reply from an orch whose button is off, or sent while the relay is down, still answers a silent ok. The board entry asks that any post to SLACK say so in that state.
   - Evidence: verbs.go:392-406; the `VerbReply` dispatch in server.go never sees `slackRelayCheck`.

2. **MEDIUM, design.** `internal/creelcli/slackbridge.go` (`RelayPostReady`). A failure to read the bridge's files refuses a post that would otherwise be stored, and the refusal names no way forward.
   - An error from `b.geometry()`, from `b.beams()` inside `hasButton` for a beam orch, from `ForwardingOn` or from `RelayLive` returns as `err`. `slackRelayCheck` (verbs.go:229-231) then fails the whole post, and nothing is stored.
   - Before this diff the post was stored. The relay itself tolerates an unreadable beam listing: it posts to an open channel (outbound.go, `channelFor` doc).
   - A `RelayLive` error only means liveness is unknown, which should leave `receiver_live` absent, not refuse the post.

3. **NIT, design.** `internal/creelengine/verbs.go:226-229` (`slackRelayCheck`). The `FormatProgram` error is swallowed with `err != nil` folded into `return nil, nil`.
   - It is unreachable for a valid store shortname, and an invalid one could never match the relay name anyway, so it skips no check.

Tool calls:
1. `slackcli/toggle.go` button rule, the `Reply` header and `checkReceiverLocked`: my context lacked whether a pair's orch can have a button, whether `Reply` goes through `post`, and how a receiver name is matched.
2. `verbs.go:370-420` (the `Reply` body) and `pool.go` `checkReceiverLocked`: needed to confirm that the reply path appends with no relay check, and that receivers match exactly.
