## Goal

```markdown
## Decision (operator, 2026-11-01)

LoomYard's own batcher template, `internal/batcher/template.yaml`, makes `wide` its active profile, for every hub and not only this one.
- `wide` is `cautious` with `budget: 300000`, the same profile this hub's `batcher.yaml` has run since 2026-10-22.
- `active:` names `wide`; `cautious` and `identity` stay in the template as profiles a hub may choose.
- The comments that name `cautious` as the template default, in the template and anywhere else that says so, follow.
- No estimator weight changes.

`docs/research/batch-budget-trial-2026-11-01/report.md` is the evidence: over 23 `wide` runs it costs no more per card and shows no quality loss.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
