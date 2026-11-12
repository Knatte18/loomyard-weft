```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every refusal, notice, next step and criterion the fix added can be reached and leads somewhere that makes progress (fix-way-forward lens).

1. MEDIUM. D8, F7's first-contact notice: "A `bridge-` channel whose creator bot's app no pin holds is left unmapped, and when its newest envelope names this hub's shortname at either end, the resync posts one notice…"
   - Summary: the notice may never fire, so first contact stalls with nothing telling the operator.
   - Evidence: the sentence just before it, added for F5, says the resync "considers only messages whose bot is this bot or whose app is the pinned app of their envelope's `from` machine". On first contact the creator's app has no pin, so no message in the channel passes that filter, and "its newest envelope" names nothing.
   - The record must say that this notice check reads the newest envelope whoever posted it. It is only used to route the notice, never to set the local orch. Without that, the D8 criterion "posts one notice in the prime orch's channel naming its app id" cannot be reached.

2. MEDIUM. D8, F13's retired-orch skip: "When the local orch is no longer a live orch of this hub, the message is skipped with one notice per episode in the bridge channel naming the orch as retired". The same applies to the resync rule "it must be a live orch of this hub".
   - Summary: the notice only fires on a channel the relay reads, and in the case F13 describes, the relay never reads the channel. Mail is lost and no one is told.
   - Evidence: D4 says "When an orch retires, its relay archives its bridge channels". The peer's next post is refused as archived, so the peer drops the channel and creates a new one (D4). The new resync rule never adds that channel, so the relay never reads it, and the skip-with-notice bullet is never reached.
   - Meanwhile the peer's posts are accepted by Slack and settled delivered (D7). The sending orch is told its mail was delivered while it is lost, and neither operator gets a notice.
   - Fix: the resync should post a notice for a channel it refuses because the named orch is not live, as F7 does for an unpinned creator. Otherwise the bullet in the per-message list is dead text.

Tool calls:
- Ran `sed -n 373,410p internal/slackengine/outbound.go` and grepped for `openOrchs`. My context lacked how the prime-channel notice route works (postDroppedLine) and which channels a relay reads, which I needed to check that F7's notice location is reachable. It is: the prime-channel route and the `[slack relay]` fallback exist.
