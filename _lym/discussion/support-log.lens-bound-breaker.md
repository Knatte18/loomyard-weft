```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's bounds, guarantees and claims about existing code. Three findings are MEDIUM and three are LOW.

**Findings**

1. **MEDIUM — D7 bound and the Constraints line on creel's sender bound: the authority grant is wider than the record says.**
   - The record says the grant covers only mail from the `SLACK` program, "which posts only text from the configured operator user id". It also says "an agent's CLI post can never sign as `SLACK`".
   - `creelengine.checkProgramClaim` (`internal/creelengine/identity.go:85`) accepts any well-formed upper-case name under the hub's shortname. There is no allowlist, so `SLACK` is open to every Go-client caller outside a pane or in an orch pane.
   - Counterexample: an agent writes a small Go program with the creel client and runs it detached with `setsid`, so it is re-parented out of its pane tree. Creel places it in no pane, accepts the claim `<sn>:SLACK`, and its `[slack] …` text reaches the prime orch with operator authority.
   - Off Linux, identity is the environment alone (`resolveFromEnvironment`), so the claim is wider still.
   - Fix: restate the bound as creel's existing program-trust bound (the 0700 lymtemp root, the same trust that `LOOM` mail already gets), and drop "only text from the configured operator". Or add a decision that reserves the `SLACK` name.

2. **MEDIUM — D5: posting to Slack is a new routing edge with no stated bound.**
   - Any strand, including a task's driver, a reviewer or a prompt-injected writer, can run `lym creel post <sn>:SLACK "<text>"`. The relay posts every message in its mailbox to the employer's Slack channel.
   - Nothing limits who may post there or how often, so this is an attack-surface finding under the rubric: it can spam the operator's phone, push repository text out to Slack, and imitate an orch. The `[<sender>]` prefix shows the real stamped sender, but that is not stated as the bound.
   - Fix: state the bound. Either the sender prefix is the only guard and any strand may post, or the relay forwards only mail from orch strands and lym programs.

3. **MEDIUM — D2: `lym slack setup`'s checks need scopes the record never names.**
   - Setup checks the channel with `conversations.info` and the operator with `users.info`. Those calls need `channels:read` or `groups:read`, and `users:read`.
   - The Constraints and the Review criterion name only `channels:history` or `groups:history` plus `chat:write`.
   - Counterexample: an app installed with exactly the named scopes fails setup with `missing_scope`, and every step after that is blocked.
   - Fix: add the read scopes, or drop those checks.

4. **LOW — D4: the cursor rule covers only messages forwarded to creel.**
   - Presence commands, the unknown-slug answer and the too-long answer post nothing to creel. If the cursor advances only "after creel answers the post's id", none of these moves it, and each poll answers the same message again.
   - Fix: state that the cursor advances past every message handled, including skipped and answered ones.

5. **LOW — D6 and D8: mail can reach the `SLACK` receiver before it exists.**
   - Creel accepts a program as a receiver only once that program has read.
   - Counterexample: on a fresh hub, `lym slack away` starts the relay and mails the orchs in the same call. The orch mirrors its next message to `<sn>:SLACK` before the relay's first `read --wait`, and creel refuses the post as an unknown receiver.
   - Fix: `away` waits until the relay is live and has read before it sets presence and sends its notices.

6. **LOW — D8: "the time the run entered that row" is not a field.**
   - `shedengine.Status` has `CurrentProducer`. The times sit only on `HistoryEntry.At` and `LastAt`. The entry time has to be taken from the history entry whose result made the row current; a resumed or folded call shifts it.
   - Batten already reads the child's status for this check (`internal/battencli/wire.go:781`), so the claim that dobby reads the child's row holds.
   - Fix: name the derivation in the record.

**Claims attacked**

| Claim | Result |
|---|---|
| D2 bound: the token reaches no agent's environment or prompt | Holds for the environment; any agent's Read tool can still open the file (LOW, noted only) |
| D3: a second start for the same hub starts nothing | Holds, given the single-instance lock |
| D3: re-exec on a replaced binary | Holds, provided the lock is taken again after exec; Go sets close-on-exec, so it is dropped for a moment and a racing start can take it |
| D4 bound: duplicate on a crash between post and cursor write | Holds |
| D4: no history forwarded on first start | Holds |
| D4: return to 5 s on a presence change | The mechanism is unstated; a change made by another process needs `internal/fswatch` (LOW) |
| D4: three hubs at 36 requests a minute at most | Holds per method and per app; the "three hubs" figure is a tally |
| D4: every message is skipped except operator top-level messages | Holds |
| D5: at-least-once delivery, ack only after `ok` | Holds; a partially failed batch re-posts the earlier messages, which is within the stated bound |
| D6: notices to orchs that are not listed | Hold; creel keeps mail for an orch by its fixed name until it is listed |
| D7 and Constraints: a program claim cannot be forged | Broken (finding 1) |
| D8: dobby can see the child's row | Holds |
| D8: the wait start can be read | Partly holds (finding 6) |
| Rejected alternative: Socket Mode spreads events across one app's connections | Not shown in the research the record cites; a stated fact with no source (LOW) |

**Tool calls**

1. Grepped for `WaitsOnOperator` users and status time fields. My context did not show which package reads the child's row, or whether status carries times.
2. Read `internal/battencli/wire.go` around line 781. I needed to confirm that the operator-wait read happens on dobby's side.
3. Grepped the time fields of `shedengine.Status` and `HistoryEntry`. I needed to know whether a row entry time exists.
4. Grepped creel's program-claim functions. I needed to locate where a program name is validated.
5. Read `internal/creelengine/identity.go` lines 80–104. I needed to know whether any upper-case program name is accepted.
