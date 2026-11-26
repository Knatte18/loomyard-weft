---
round: 1
focus:
  - "Own-post recognition (internal/slackengine/bridgein.go isOwnPost, load, handle, resync, authored): does the production Slack client fill Identity.BotID from auth.test and Message.BotID on every history and replies read, and do the fakes match the shape Slack returns, so the fix works outside the tests?"
  - "Own-post recognition as a method: is there any other read path in internal/slackengine (threads, resync, newest-envelope checks, send-side lookups of an earlier post) that still identifies this machine's posts by user id alone?"
  - "botIdentity (internal/slackengine/keeper.go): what happens when auth.test returns an empty user id or bot id, and is the whole identity cached and compared correctly at each former botUser caller?"
  - "Mutation survivor internal/slackengine/bridgein.go:109 (negated comparison in isOwnPost): could a test assert what that comparison decides, for instance a message with no bot id or no user id from someone else, or an identity lacking one of the two ids?"
  - "Relay command parsing (inbound.go handle, peercommand.go relayCommandWords and parsePeerCommand, bridgein.go handleOperator): does every `lym` command and every non-command text, in orch channels, threads and bridge channels, reach the destination the amended board entry expects, including Slack-escaped text, extra words and unknown verbs?"
  - "Old command forms: does any answer, notice, way-forward line, doc, spec, stencil or contract anywhere in the repository still tell the operator to type a bare `approve`, `revoke`, `status`, `away` or `back` in Slack?"
  - "docs/slack-app.md as the diff leaves it: read the pairing steps and the new `Relay commands` section, including where the closing line about the manifest now falls and whether each statement matches the code."
  - "_lym/landing/summary.md: check each claim against the diff and tests, and check the `two-way` Door and the `slack` Blast radius against everything the diff can break, including the creelengine change, the changed operator command syntax and any record the resync now writes."
---

Seed focus drawn from the amended board entry, the commit messages, the non-test diff and the mutation report.
The entries name places to look and carry no verdict.
