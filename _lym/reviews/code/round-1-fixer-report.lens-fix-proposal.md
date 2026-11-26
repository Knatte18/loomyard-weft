```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3, all three applied as proposed.

no findings

- **F1:** the new overview wording matches `inbound.handle`, which answers the three one-word presence commands, and `parsePeerCommand`, which handles `lym approve` and `lym revoke`. A malformed form such as a bare `lym approve` is forwarded, but the sentence names the commands rather than every input shape.
- **F2:** the new row does decide the user-id clause. With the user comparison negated or the clause removed, the bot-id clause is false (`BotID` is empty). `authored` then returns false at `message.BotID == ""`, so the channel is not adopted and the row fails.
- **F3:** the new row does decide `len(words) == 1`. Without that condition, `lym back now` runs `back`: nothing is posted and the presence stays `terminal`, against the expected post and `slack`. `parsePeerCommand` does not match the text, so the row expects the right forwarding.

Outside my lens and not caused by this fix: `docs/overview.md` still says "The operator's message in Slack sets the presence to `slack`", although the `lym status` test row leaves the presence at `terminal`.

Further tool calls: none.
