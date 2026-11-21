## Goal

```markdown
Imported from [issue #161](https://github.com/Knatte18/loomyard/issues/161).

## What was seen

A task run's Finalize halted with `landing stop: GitHub refused to merge pull request ...: Pull Request is not mergeable` and a way forward naming a second approval or a required check.
Read a minute later, the pull request showed `mergeable: MERGEABLE`, `mergeStateStatus: CLEAN`, no required checks and no reviews; GitHub had not yet computed mergeability (`UNKNOWN`) when Finalize asked.

## Workaround

The parent re-reads the pull request with `gh pr view --json mergeable,mergeStateStatus` and runs `lym loom resume` once it reads `MERGEABLE`.

## Expected

Finalize treats `mergeable: UNKNOWN` as pending: it polls the pull request for a bounded time before merging, and halts only on a definite refusal, naming the actual cause.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - merge-state-read-failure-untested (class design, severity MEDIUM, open in rounds 1)
  - uncomputed-mergeability-halt-advice (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
