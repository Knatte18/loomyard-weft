```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (F1, F2, F3) by building cases where the proposal itself goes wrong.

Findings:

1. **MEDIUM.** D5's added text: "The wait backs off, so a persistently refused invite costs one local read and one Slack call per wait."
   This cost bound came over from the F2 proposal, and it holds only if a pass whose invite fails counts as idle.
   `loopWait.afterActivity` (`internal/slackengine/backoff.go:52`) resets the wait to the shared floor after a pass that did something.
   Only `afterIdle` doubles the wait toward `waitCeiling`.
   The record never says how a pass with a failed invite counts.
   If it counts as activity, the wait stays at the floor, and a refused invite repeats one Slack call per floor interval for each open channel, not once per backed-off wait.
   Each beam with a refused invite adds its own call.
   The same gap affects the owed-read retry in D2, where a failing read would retry at the floor.
   D2's "until it stands" and the Constraints line's "no new loop" both assume the wait backs off.

2. **LOW.** D5's added text: "the inbound filter (D7) uses the operator that read last gave."
   Case: the operator runs `lym slack setup --operator U2`, and U2 posts in the public prime channel before the next wake from the outbound wait, which can be up to sixty seconds away.
   A hint read or an owed read runs in that window with the old operator in the filter.
   D7 skips "anyone's but the configured operator's", so U2's message is skipped and the cursor moves past it ("Cursors only move forward", D2).
   After the wake, nothing reads that message again.
   The proposal said the change takes effect "at that wake" and left this window open.
   Reading `setup.json` at every wake, file events included, would close it.

3. **LOW.** D1's added text: "It is the notice D3 mails on its ensure path, with D3's way forward, and one episode covers both."
   D3's ensure-path notice goes to "the ensuring hub's orch", and any hub's relay can make an ensure.
   D1's new notice goes to the holder hub's orch.
   When the ensuring hub is not the holder's hub, these are two notices to two orchs, so "one episode covers both" holds only for the holder's own hub.
   The sentence reads as if the two notices are deduplicated machine-wide.

No further tool calls made.
