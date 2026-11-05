```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fix carried out as written, and report each concrete case where a proposal is wrong or breaks something else in the artifact.

1. **MEDIUM**: card 07 hits a persistent presence error and gets stuck for good with no answer to the operator.
   - **Changed text:** `_lym/plan/07-slackengine-inbound.md`, the presence-command bullet, "`SetPresence` returning `changed` false with an error, the presence not written, makes the poll return the error with the cursor left before the message, and the relay retries with its backoff…; `status` does the same when `ReadPresence` errors". Also the new row "a `status` over a presence file holding an unknown value gets no answer, returns the error and leaves the cursor".
   - **Case:** `presence.json` holds an unknown value. Card 04:51 says "an unknown stored value is an error naming the file". The operator, who is away, texts `status`. The poll returns the error with the cursor left, and the next poll meets the same file and the same error. This repeats on every poll, so no retry ever succeeds. Every later Slack message, routed mail included, waits behind it, and the operator gets no Slack answer saying why. `away` and `back` over the same file get stuck the same way if `WritePresence` reads the old value to report `changed`.
   - **Why the proposal is wrong here:** leaving the cursor and retrying with backoff fits only a transient failure, such as a failed write. A `status` read changes nothing, so a retry gains nothing.
   - **Better outcome:** for an error that persists, answer in Slack naming the error and the file, and move the cursor.
   - **Evidence:** card 04 line 51 and its `TestPresence` row "an unknown stored value is an error". Card 07 step 3's own rule for failed posts gives up after 5 minutes, but the presence branch has no such bound.

2. **LOW**: card 09's lifetime rule still allows unlimited respawns, as long as each relay lives at least 10 minutes.
   - **Changed text:** `_lym/plan/09-slackengine-relay.md`, "a lifetime at or over it sets `StartFailures` to 0 and counts none".
   - **Case:** a relay is killed every 30 minutes, for instance by a leak that triggers the OOM killer. Each due `whileAway` start reads a lifetime over 10 minutes, resets the count to 0 and spawns again. It never answers `gave-up`, so the loop runs for as long as presence stays `slack`.
   - **Evidence:** `pattern/PATTERN-child-processes.md:16` says "A retry loop around a real spawn caps attempt COUNT, not only elapsed time". The count is now reset by elapsed lifetime, which is the time-only bound that rule rejects. The rate is bounded (at most one spawn per 10 minutes), not the count.
   - **What to settle:** the card should state this as an accepted bound, or cap total spawns per episode.

The other proposals hold up: F2's `changed` split, F3, F4 (1 to 16 minutes stays right under F1), F5, F6 and F7. I traced each new and rewritten TestEnsureRelay row through the start sequence and none contradicts the rule.

Tool calls:
- One grep of `pattern/PATTERN-child-processes.md` for the retry-cap line and of `_lym/plan/04-slackengine-files.md` for the presence contract. My context did not hold the pattern's exact wording or card 04's `ReadPresence` error rule.
