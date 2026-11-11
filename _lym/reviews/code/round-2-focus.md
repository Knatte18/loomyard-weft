---
round: 2
focus:
  - "In internal/slackengine/relay.go, settle whether RunRelay ends when the forwarding list is empty after a retire pass, and that a relay started with no button still behaves as the tests expect."
  - "In internal/slackengine/keeper.go, settle whether channelKeeper.retire drops forwarding entries of retired beams that have no channel-map entry or an already archived one, and retries a failed dropForwarding on a later pass."
  - "In internal/slackcli/wiring.go, settle whether worktreeOrch (prime, beam, pair, unlisted worktree) and orchProject (empty slug, project entry, none, unreadable board) are reached by tests that do not fake those seams."
  - "In pattern/PATTERN-daemon-wakeups.md, settle whether the Slack relay bullet names the relay's wake watch as the code leaves it."
---

Focus lists where fixes land; the review still covers the whole range.
