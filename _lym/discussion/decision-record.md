## Goal

```markdown
## What was seen

2026-11-19: `lym creel post ly:SLACK "<text>"` answered `{"ok":true}` (message 2035) while the operator had disconnected Slack.
`lym slack status` showed `relay_live: false`, `connected: false` and an empty `forwarding` list, so the post reached no channel, and the sender was not told.
The orch believed the result was on Slack.

## Idea

A post to SLACK while no relay runs, or while the sending orch has its slack button off, should say so:
refuse with the state and a way forward (`lym slack start`, or press the button), or answer ok with a `delivered: false` field and the reason.
The sender then reports the result in its pane instead.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - relay-receiver-guard-unpinned (class design, severity MEDIUM, open in rounds 1)
  - reply-to-slack-mail-skips-relay-check (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
