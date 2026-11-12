```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard, guarantee and claim about existing code in the decision record. I made 2 tool calls.

**Findings**

1. **BLOCKING — D8, dedupe and reply-to key.** "It answers an id it already holds for the same machine and sending id without storing again."
   - Counterexample: hubs `ly` and `bx` on neve each mint their own id 5 and both post to `osl-1046::ly:orch`. Each hub's pool mints ids with its own counter (`pool.newID` in `internal/creelengine/fold.go`).
   - The receiver keys on (machine, sending id), so it takes the second message as already held and drops it silently, or treats it as an id clash.
   - The D8 reply-to check ("this hub sent that id to that sender") has the same gap.
   - The cross-hub path keys on the hub path plus the id. The bridge key needs the sending hub too: the shortname from the `from` name, or an explicit hub field in the envelope.

2. **BLOCKING — D8, the held cursor.** "Otherwise it holds the cursor before it… An approval or a fixed pin delivers every held message."
   - Counterexample: a bridge channel is public. Any workspace member can join it, add their own bot, and post `[lym-bridge] from=osl-1046::ly:orch to=neve::ly:orch id=1`.
   - The app does not match osl-1046's pin, so the cursor holds. Every later real message from the approved osl-1046 app stays undelivered.
   - The only way out is approving the impostor's app. A "fixed pin" cannot help, because the pin is already correct.
   - Fix: when the `from` machine is already pinned and the app does not match, skip the message with a notice instead of holding. Hold only while the `from` machine has no pin.

3. **MEDIUM — D6, approval when there is one candidate.** "One candidate is pinned."
   - Counterexample: osl-1046 has not created its app yet. A workspace member creates `lym-osl-1046` and invites neve's bot to a `bridge-…` channel. That channel is admitted through the D8 routing and resync. The member posts an envelope, and neve's relay asks the operator to `approve osl-1046`.
   - `approve osl-1046` finds the single impostor and pins it.
   - The operator has nothing to check the app id against. Each relay should show its own app id, for example in the `status` answer, so the operator can compare before approving.
   - The bound "An approved app is trusted to name its sender" leaves this trust-on-first-use path unstated.

4. **MEDIUM — D3, bound wording.** "No agent pane can make that claim" is false.
   - `internal/creelengine/doc.go` (Sender identity) says "a program name is accepted from an orch strand's pane only". So the orch pane, the very strand that uses the bridge, can claim `<shortname>:SLACK` through the Go client.
   - From there it can call the deliver verb and forge trusted `(other machine, trusted orch)` mail to itself or another orch of the hub.
   - The bound should say this: the orch's pane, through the Go client, and any process outside a pane.

5. **MEDIUM — D8, unbounded bridge-map growth.** The holder routing and the users.conversations resync add any unmapped `bridge-` channel whose envelope names this machine.
   - Any member can invite the bot into many such channels. Each one joins the map for good and is polled every bridge poll. That costs conversations.history calls against Slack's rate limits and repeats the notices.
   - No bound limits additions to channels made by an approved peer's or this machine's bot. The creator check that D4 applies to `name_taken` is missing here.

6. **LOW — D4, "without listing the workspace".** The side that sends second, when its map lacks the channel and no event or resync has come yet, hits `name_taken`.
   - Finding the existing channel's id then goes through `FindChannel`, which is conversations.list over the whole workspace.

7. **LOW — D4, name collisions.** Sanitizing `:` to `-` and joining the ends with `--` is not injective.
   - Machine `neve` with orch `ly:x:orch` gives the end `neve-ly-x-orch`. A machine `neve-ly` with orch `x:orch` gives the same end.
   - Machine names may also hold `--`. A collision with one's own pair is caught only by the creator check.

8. **LOW — D6 and Open risks, machine rename.** The record says a renamed machine needs re-approval, but approval matches bot users named `lym-<machine>`.
   - `lym machine name` does not rename the Slack app, so `approve <new name>` finds no candidate. Only the `<app id>` form works.
   - D1 also refuses only the current name, so a post to this machine's former name (`machinename.Names`) is routed as a remote machine.

9. **LOW — D11, existing-code claim.** "The relay already calls … bots.info" is not right. `BotAppID` is called only by `lym slack login` (`internal/slackcli/cli.go`), not by the relay.
   - The evidence list also leaves out users.list, which D6 relies on.

10. **LOW — D8, resync.** "Read from its newest envelope" fails when the newest message is the operator's or this bot's own and carries no envelope. That channel is never mapped. The resync should take the newest message that holds an envelope.

**Claims attacked, held or broken**

| Decision | Claim | Result |
|---|---|---|
| D1 | Only own-name refusal | Broken for former names (LOW) |
| D2 | A pair's orch is refused | Held, through D3's button seam |
| D3 | Program-claim bound | Broken (MEDIUM) |
| D3 | Refusal at post, before journaling | Held |
| D3 | Hour and return parity | Held |
| D4 | Both sides compute the same name | Held |
| D4 | Name is injective | Broken (LOW) |
| D4 | No workspace listing | Broken (LOW) |
| D4 | Creator check on `name_taken` | Held |
| D4 | Refused-invite path | Held |
| D5 | Unparsed first line is skipped | Held |
| D6 | Pin defeats impostors | Broken when there is one candidate (MEDIUM) |
| D6 | Commands only from the operator | Held |
| D7 | At least once with no repost | Held, given receiver dedupe, but that dedupe is itself broken (1) |
| D8 | Dedupe and reply-to keys | Broken (BLOCKING) |
| D8 | Hold until approval | Broken, DoS through an impostor post (BLOCKING) |
| D8 | Map admission | Unbounded (MEDIUM) |
| D8 | Resync | Broken in an edge case (LOW) |
| D8 | Button hold | Held |
| D10 | One-second floor and backoff | Held |
| D10 | Poll carries mail without events | Held |
| D11 | Scopes | Held |
| D11 | Relay calls bots.info | Broken (LOW) |

**Tool calls**

- Read `internal/creelengine/doc.go` lines 70–92 and grepped it for id minting: the record's D3 bound rests on the program-claim rule, which my context did not quote.
- Grepped creelengine for `newID` and the receive.go dedupe tuple: to check whether creel ids are per hub, which my context did not show.
