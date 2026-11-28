## Goal

```markdown
## Seen

The landing that has the darn writer ask before an entry's measurement gate calls (`ef9062d91`) closed Code-Review over two accepted-circling findings on the new stencil rule:

- `measurement-rule-no-parent-path-drops-deliverable`: with no parent session, the measurements the entry asks for are never produced.
- `ungated-subagents-bypass-measurement-rule`: the gated fragment has no sentence of its own barring subagents from measurement calls.

## Fix

- Give the rule a path for a run with no parent, so the entry's measurements are still produced or the run stops with a way forward.
- Add a sentence to the gated fragment that bars subagents from measurement calls.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - F2 (class decision, severity LOW, open in rounds 1)
  - closed-scope-measure-stop-loops-on-resume (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
