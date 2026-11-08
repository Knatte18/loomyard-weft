---
round: 2
focus:
  - "Card order and granularity around the credential record: if the engine's credential record (`Credentials`, `ReadCredentials`, `WriteCredentials`, `RequireConfigured`) and the two-token login CLI now sit on separate cards, check that each card's Uses, Targets and test rows are consistent, that no card depends on a later one, and that no old `ReadToken`/`WriteToken` reference survives in a card that runs before the record lands."
  - "Any card whose Intent was rewritten to add or change a recovery, guard or refusal branch (outbound `postTo` recovery for the stale and dropped lines, inbound `answer`, the keeper's `LiveHubs` handling, the missing-setup refusal): settle whether each prescribed branch has a named input and a named test row that reaches it, and whether an upstream check rejects that input."
  - "Plan-wide wording that names cards or symbols by number or by old name (cross-references between cards, `doc.go` prescriptions for `internal/slackengine` and `internal/slackcli`): settle whether each reference matches the card that now carries the work after any renumbering."
---

The focus targets the places a round-1 fix could have changed text no fresh reviewer has read.
