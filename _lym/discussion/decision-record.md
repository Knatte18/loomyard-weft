## Goal

```markdown
## Seen (operator, 2026-11-27)

The operator tried the agent button in reed's status bar for the first time.

1. It refused a name with spaces without saying what a valid name is.
2. The agent it spawned knew nothing of lym: no skills, no commands, no idea what it is, which hub, worktree or task it serves, or what it is for.

## Task

- Name the rule for the agent's name when asking for it; on a refusal name the rule and ask again, or turn spaces into dashes.
- Spawn the agent with an opening stencil that says what it is, which hub, worktree and task (or the prime) it sits in and what it is for,
  and which skills to load for that context, such as `ly:cli`, `ly:board` and the scribe skills.
  Give it the same instructions, guards and skills any other lym session of its engine gets.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - agent-parent-directive-contradicts-stencil (class consistency, severity MEDIUM, open in rounds 1)
  - agent-session-has-no-end-of-life (class design, severity MEDIUM, open in rounds 1)
  - agent-verb-untested (class design, severity MEDIUM, open in rounds 1)
  - open-role-normalization-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
