## Goal

```markdown
## Seen (orch, 2026-11-17)

The decision record's decisions carry IDs (`internal/discussionparser/decisionids.go`), but no review checks a diff for work no decision asked for.
Source: the survey F17 (`orchestrator-survey-ideas`), from Spec Kit's `/converge`, which classifies gaps as missing, partial, contradicting or unrequested.

## Idea

- A lens in code review that maps each change in the diff to a decision ID and flags a change that traces to none as unrequested scope.
- It also flags a decision no change builds.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - code-scope-writer-claimed-decision-source (class design, severity MEDIUM, open in rounds 1)
  - dotnet-hub-review-code-fan-omits-code-scope (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
