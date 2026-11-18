## Goal

```markdown
An agent filed issue #136 with `lym selfreport create "<title>" -b .scratch/selfreport-missing-brief.md`.
`-b` takes the body text, so the issue body became the path and the report itself was lost.

Idea: give `lym selfreport create` a `--body-file <path>` flag (with `-` for stdin), and refuse or warn when `-b` text is a single line naming an existing file.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
