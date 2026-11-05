## Goal

```markdown
## Seen (2026-11-05)

After `raddle-recipe` (`257ab79a`), a Sketch becomes a task only through `lym board promote <slug> --recipe raddle`; a plain `lym board promote <slug>` of a Sketch is refused, naming that flag as the way forward.
Since raddle is the one recipe a Sketch may take, the flag only adds typing, and the operator finds the command clumsy.

## Wanted

- A plain `lym board promote <slug>` of a Sketch (a discussed note with a stored record) promotes it with recipe `raddle`, as if `--recipe raddle` were given.
- `--recipe raddle` stays accepted; any other recipe on a Sketch is still refused, naming the plain promote as the way forward.
- The handle form works too: `lym board promote S3`.
- The refusal texts, the `ly:board` skill and the docs that name `--recipe raddle` as the required form are updated.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
