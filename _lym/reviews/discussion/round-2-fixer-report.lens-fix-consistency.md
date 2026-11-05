```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the decision record's sibling decisions, criteria and cross-references (fix-consistency lens).

1. **LOW: the Scope section doesn't list the new PATTERN change.**
   - Changed text: `## PATTERN changes`, the new `PATTERN-reap-children — changes:` bullet, and D3's double-fork bullet.
   - Scope's In list says only "One new PATTERN entry for the Slack boundary."
   - The record now also changes `PATTERN-reap-children`, and Scope does not mention that change.

2. **LOW: D2 relies on a D4 path that only runs at start-up.**
   - Changed text: D2's new line "a `lym slack setup` reaches a live relay at its next poll, and a changed channel takes D4's first-start path."
   - D4 defines that path as a step taken before the first poll: "A first start with no cursor for the configured channel writes the cursor … before its first poll."
   - A live relay that sees a new channel mid-run is not starting, so D4 doesn't say it writes the new channel's cursor before it reads that channel.
   - The D4 criterion "after a `lym slack setup` changes the channel, the relay forwards none of the new channel's history" depends on that missing step.

3. **LOW: D5 gives two different waits after a 429.**
   - Changed text: D5's new line "it waits outside the handler, with the poll's backoff or a `429`'s `Retry-After`".
   - A later D5 bullet still says "a failed transport, a `429` or a `5xx` counts toward no cap and is retried with the backoff."
   - D4 also says "A `429` waits out Slack's `Retry-After`", so the old bullet now conflicts with both.

4. **LOW: D4's Bound doesn't cover messages held behind a refused post.**
   - Changed text: D4's new line "retried with the poll's backoff, without limit, so later messages wait behind it."
   - D4's Bound lists the delays the operator's messages can meet, but not that a refused creel post holds every later message without limit.
   - Those held messages include the presence texts `away`, `back` and `status` (D6), which D4 says go to no orch.

Further tool calls: none; my context already held everything this lens needed.
