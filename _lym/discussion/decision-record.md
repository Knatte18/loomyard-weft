## Goal

```markdown
Imported from [issue #86](https://github.com/Knatte18/loomyard/issues/86).

## Seen

In hub `dop`, the operator finds the parent orch's review of a run's PR very useful, since the orch knows what the run needs.
Today the orch reviews a PR only when the operator tells the run's PR-Review reviewer to ask its parent orch for a review, as for PR 27 and PR 28, or asks the orch directly, as for PR 25 and PR 26.
Nothing is broken; the operator has to ask each time.

## Change

A setting, per hub and per task (a loom key with a `run_settings` override, beside the PR-Review perch's other keys), that makes the PR-Review step ask the parent orch for a review on its own when the run's PR is open.
The request names the slug, the PR URL and the branch.
With the setting on, the run waits at PR-Review until the orch answers satisfied or sends findings; findings are handled as the reviewer handles the orch's findings in the flow the operator triggers by hand today.
With the setting off, PR-Review runs as now.
The orch role gets the instruction for answering such a request.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - pr-review-does-not-wait-on-orch-answer (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
