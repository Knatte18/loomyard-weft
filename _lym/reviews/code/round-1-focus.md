---
round: 1
focus:
  - "internal/slackengine/inbound.go: does any production code or other reader still match the removed `[slack] ` body prefix (grep the repo for `[slack]`, `forwardPrefix`, `HasPrefix` on SLACK mail)? The board entry asks to check that no reader keys on it."
  - "contracts/orchrole/role-slack.md: does the new rule for telling the operator's words from `[slack presence]` and `[slack relay]` notices cover every non-operator notice the engine still sends, and is the old prefix rule fully gone?"
  - "internal/slackengine/inbound_test.go and relay_test.go: do the edited tests still assert the forwarded body without prefix, and is any test now a duplicate of another covering the same behavior?"
  - "docs/overview.md and internal/slackengine/doc.go: does any other doc or comment still describe the `[slack]` prefix?"
---

Single small commit; the main risks are a missed reader of the prefix and a role-file rule that misclassifies other SLACK notices.
