## Goal

```markdown
Imported from [issue #110](https://github.com/Knatte18/loomyard/issues/110).

What happened: during a burler plan round the harness moved the session's working directory to the weft's `_lym/plan`. `lym shuttle await-forks` then failed with "cwd is outside the recorded fabric anchor subtree", because it accepts only the anchored directory itself rather than anything inside the anchor subtree.

Why abnormal: the message says "subtree", yet a directory inside the weft's own `_lym/plan` is refused. The session cannot control the harness's cwd changes. The self-check fell back to taking each fork's result as it arrived, across four turns.

What to do: make `lym shuttle await-forks` accept a cwd anywhere under the recorded anchor, or resolve the anchor from the weft root. Otherwise make the error name the required directory.

Provenance: task slug adopt-worker-record; note burler-plan-r2.md.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - await-forks-dispatch-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
