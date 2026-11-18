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

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - body-flag-names-raw-literals (class consistency, severity BLOCKING, open in rounds 1)
  - padded-path-trim-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
