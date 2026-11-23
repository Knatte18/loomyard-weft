## Goal

```markdown
## Seen

`internal/hardenshed/template.yaml` sets `reviewer: [opus, fable, sonnet]` and `fixer: [opus, fable, sonnet]`, so every hardener campaign rotates through the three models round by round.
The rotation came from the hardener's own run (`7346d6a8c`, 2026-11-19); the operator never chose it.
Fable is too dear for a default, and sonnet found plenty in the Crucible campaigns: what a round finds rests on its seeded focus, not on the model.

## Decision

Operator, 2026-11-22:

- The template default for `reviewer` and `fixer` is `sonnet[high]` alone, with no rotation.
- Which models a campaign uses, a rotation included, is the operator's choice per run, set in the task's `run_settings` under `hardener`.
- The template comments and any doc or pattern text that describes the rotation as the default say so.

Check that `run_settings` takes `hardener.reviewer` and `hardener.fixer` as lists, and fix it if not.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): A one-entry reviewer list passes preflight

Decision: hardenshed's preflight accepts a reviewer list of one entry; the refusal of two neighbouring equal entries stays for lists of two or more.

Rationale: The operator set the template default to sonnet[high] alone with no rotation (2026-11-22), which the one-entry refusal would reject.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
