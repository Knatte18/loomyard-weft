```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the F1–F3 change set.

Findings left unchecked: none. F1, F2 and F3 were each checked.

no findings

- F1: the searches of docs and package comments found no other site that still says the relay answers a bare command in the channel. `docs/slack-app.md`, `internal/slackengine/doc.go` and the `inbound.handle` comment already use the `lym` forms. The bare names in the `docs/overview.md` verb lists are `lym slack` subcommands.
- F2: `isOwnPost` has its callers in `handle` and `authored`. The new row has `UBOT` and no bot id, so only the user-id comparison can mark the post as authored, since no pin exists for this machine's own name.
- F3: presence words reach their handlers only through the single `len(words) == 1` condition in `inbound.handle`, so one row covers all of them.

Tool calls:
1. `git grep` for bare relay command words in docs and non-test sources, because my context held only the sites the review named.
2. `grep` for `isOwnPost`, `botIdentity` and the bot-id fixtures in `internal/slackengine`, to list the callers and fixtures.
3. `grep` for channel-command wording in the slack, creel and reed packages and `docs/slack-app.md`, because the first search could miss other phrasings.
